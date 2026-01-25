# Engineering-Portfolio

This repository contains real-world system design case studies based on production experience.
Each document focuses on **technical decision-making**, **trade-offs**, and **failure-aware architecture design** rather than implementation details.

The goal of this repository is to demonstrate how I approach complex backend and distributed system problems as a Staff-level Individual Contributor.

---

## How to Read

Each case study follows a consistent decision-driven structure:

1. Context & Constraints
2. Design Options Considered
3. Decision & Rationale
4. Finalized Architecture
5. Trade-offs & Risks
6. Outcome
7. What I’d Change Next Time

---

## Case Studies

### Payments & Monetization
- [In-App Purchase Verification System](./cases/iap-verification/README.md)
  - Stateless verification service
  - WebSocket-based stateful backend isolation
  - Transactional Outbox Pattern
  - Asynchronous entitlement delivery via SQS

### (Planned)
- Zero-downtime deployment for stateful WebSocket services
- Global traffic routing and latency trade-offs
- Metrics aggregation and observability platform design
- Incident-driven architecture evolution
