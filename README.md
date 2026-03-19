# Engineering Portfolio

This repository documents real-world engineering experience across three dimensions:
**technical decision-making**, **production failure analysis**, and **engineering leadership**.

Each document is written to reflect how I think about problems — not just what was built,
but why specific approaches were chosen, what trade-offs were accepted, and what I learned.

---

## Case Studies
> Architecture decisions and system design trade-offs from production systems.
> Each case follows a decision-driven structure: problem → options evaluated → decision → outcome.

| Title | Link |
|-------|------|
| WebSocket Server Zero Downtime Deployment | [README](./cases/websocket-server-zero-downtime-deployment/README.md) |
| In-App Purchase Verification System | [README](./cases/iap-verification/README.md) |
| Horizontal Scaling of WebSocket Server Using Redis Pub/Sub | [README](./cases/redis-pubsub-scaling/README.md) |
| PLP Multi-Region Expansion | [README](./cases/plp-multi-region-expansion/README.md) |

**Planned**
- Graceful shutdown using Kubernetes
- BFF pattern for cross-platform delivery
- Lazy loading UI modules
- Multi-user chat architecture trade-offs

---

## Incident Reports
> Post-mortems from production failures.
> Each report documents how the failure was detected, diagnosed across layers, and permanently resolved.

| Title | Link |
|-------|------|
| JVM Memory Leak via Unclosed Kotlin Channel in WebSocket Service | [README](./incidents/jvm-memory-leak-websocket/README.md) |

**Planned**
- Circuit breaker activation during iPhone 15 pre-order traffic spike

---

## Leadership
> Engineering leadership experience: team building, process design, and culture.
> Focused on decisions made at the team level and their measurable outcomes.

| Title | Link |
|-------|------|
|  | |

**Planned**
- Building and Leading a 5-Person Cross-Cultural Engineering Team
- Introducing AI-assisted code review into CI pipeline

---

## How to Read

### Case Studies
1. Overview
2. The Problem
3. Design Options & Evaluation
4. The Solution
5. Trade-offs & Risks
6. The Outcome
7. Retrospective

### Incident Reports
1. Summary
2. Timeline
3. Root Cause Analysis
4. Resolution
5. Prevention
6. Retrospective

### Leadership
1. Context
2. The Challenge
3. Approach
4. What Changed
5. Retrospective
