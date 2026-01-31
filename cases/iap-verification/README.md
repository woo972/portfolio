## Overview
Almigo is a 0-to-1 conversational AI platform featuring a digital human interface. While the core interaction relies on stateful, long-lived WebSocket sessions, the In-App Purchase (IAP) verification must interact with stateless HTTP-based external providers (Apple/Google).
## The Problem
Integrating a mission-critical financial transaction (IAP) with a stateful real-time server created a high-risk dependency. If the main backend restarted during a verification callback, the purchase result could be lost, leading to failed product delivery and customer dissatisfaction.
### Key Constraints
- Hybrid Protocols: Core service is WebSocket-based; IAP triggers are HTTP-based.
- Provider Asynchrony: Google uses synchronous verification, whereas Apple relies on asynchronous Webhooks.
- Data Durability: Purchase results must never be lost, even during server downtime.
- Auditability: Every step of the payment history must be persisted for post-audits.
## Design Options & Evaluation
### 1. Architectural Isolation
#### Option A: Monolithic Integration
- Pros: Simple maintenance.
- Cons: High risk of data loss during stateful server restarts.
#### Option B (Selected): Dedicated IAP Service
- Pros: Decouples payment lifecycle from WebSocket state.
- Cons: Increased operational overhead.
### 2. Delivery Mechanism (IAP → Main Backend)
#### Option A: Synchronous Polling
- Pros: Conceptual simplicity.
- Cons: Blocking calls; cannot guarantee delivery if the receiver is down.
#### Option B (Selected): Asynchronous Messaging
- Pros: Ensures delivery once the Main Backend becomes available.
- Cons: Introduces Eventual Consistency.
#### 3. Atomic Consistency (DB vs Message Broker)
We needed to ensure that saving an audit log to the DB and publishing a message to the broker happened as a single atomic unit
#### Option A: Two-Phase Commit (2PC)
- Cons: Not supported by modern message brokers and creates tight coupling.
#### Option B (Selected): Transactional Outbox Pattern
1. Verification results are saved to the DB.
2. An "Outbox" record is created within the same database transaction.
3. A Relay process polls the Outbox and publishes the message.
4. The Main Backend processes the message (Idempotency applied).

## Sequence Flow
![sequence](./sequence.png)

## Trade-Offs & Risks
- Increased Complexity: Requires managing an additional microservice and a background relay worker.
- Eventual Consistency: A slight delay exists between payment verification and user entitlement (granting goods).
- Observability: Failures in an asynchronous pipeline are harder to track than immediate synchronous errors.
## The Outcome
- Zero Data Loss: Successfully prevented purchase result loss during main backend deployments or crashes.
- Independent Scalability: The IAP service can now scale according to transaction volume, independent of the WebSocket traffic.
- Fault Tolerance: If the main backend is offline, the IAP service continues to collect and store results, which are processed automatically upon backend recovery.
## Retrospective
- WorkMonitoring: I would prioritize building a custom dashboard for the message broker to gain better visibility into message lag and retry counts.
- Testing: Developing an E2E test suite that mocks Apple/Google Webhook callbacks would further validate the robustness of the Outbox Relay under load.
