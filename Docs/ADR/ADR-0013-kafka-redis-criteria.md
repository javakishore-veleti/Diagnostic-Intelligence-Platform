# ADR-0013: Kafka and Redis introduction criteria

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.13
- **Requirements:** PRD §6 non-goals, §13.3 events, §15 baseline

## Context

PRD §6 forbids introducing Kafka before a verified requirement justifies it, and §15 lists Redis as Phase 3 "only when use cases justify it". Both are easy to add and hard to remove, and both cost memory in a local stack that must run on one laptop.

The failure mode this ADR guards against is not adding them — it is adding them because the architecture diagram looked incomplete without them.

## Decision

**Neither is part of the baseline. Each has explicit entry criteria, and adding one requires an ADR citing which criterion was met, with evidence.**

### Redis enters when at least one holds

1. A measured p95 breach of NFR-PERF-001 or NFR-PERF-002 on a read path, where profiling shows repeated identical work that a cache eliminates. *Measured, not anticipated.*
2. Session or conversation state must be shared across more than one application instance.
3. Rate limiting or idempotency-key storage needs to be shared across instances.

Until then: in-process caching with a bounded size, and PostgreSQL for anything that must survive a restart.

### Kafka enters when at least one holds

1. A consumer needs to **replay** history it did not receive live.
2. An event has **two or more independent consumers** with different failure and retry characteristics.
3. Processing must continue **asynchronously** past a request's latency budget, and losing the work on restart is unacceptable.
4. A genuine backpressure problem exists that a database-backed queue cannot absorb.

Until then: the **transactional outbox** table plus an in-process dispatcher. This is not a compromise — it delivers the atomicity guarantee PRD §13.3 actually asks for, and every event keeps the full envelope (event ID, type, schema version, aggregate, occurred-at, recorded-at, producer, correlation, causation, trace context). Consumers are idempotent from day one.

**The migration path is deliberate.** Because events are already enveloped, versioned, and outbox-persisted, introducing Kafka later means changing the dispatcher, not the producers or the event contracts. The cost of deferring is therefore close to zero — which is exactly why deferring is correct.

## Consequences

- The local stack stays two containers instead of five, and starts fast enough that an unattended verification loop is practical.
- Event-driven behaviour is exercised from Phase 1 through the outbox, so the eventual Kafka introduction is a transport swap against tested contracts.
- "We might need it later" is not a criterion. If neither list is satisfied, the answer is no.

## Revisit when

A specific criterion above is met, with the measurement or requirement that demonstrates it.
