# ADR-0004: Synchronous API client approach and resilience library

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.4
- **Requirements:** PRD §13.1 API conventions, NFR-PERF-001, §17.2 availability

## Context

Contracts are source-controlled OpenAPI 3.1 documents owned by their capability (PRD §13.1). Clients may be generated but the contract stays capability-owned. Under ADR-0001 most Phase 1 calls are in-process, so this decision is mostly about establishing the convention before it is needed — and about not building HTTP hops that do not yet exist.

## Options

- Hand-written `RestClient` calls — drift from the contract, silently.
- OpenFeign — declarative, but an extra abstraction and slower-moving with current Spring versions.
- **Generated clients from the OpenAPI document** — the contract is the source, the client cannot drift.

## Decision

**Generate clients from each capability's OpenAPI document** (openapi-generator, `RestClient`-based) into a build module that is regenerated during the build and never hand-edited. For the browser, generate TypeScript clients into `@dip/api-clients` for the Angular workspace.

**Resilience4j** provides timeouts, retry, circuit breaking, and bulkheads. Every outbound call declares an explicit timeout — there is no default-infinite call anywhere.

Three rules that matter more than the library choice:

1. **Errors deserialize to RFC 9457 problem details** (PRD §13.1). One error type across every client, so callers handle failure uniformly.
2. **Correlation ID propagates** on every call, and is attached to logs and spans (PRD §19.1).
3. **Retry only idempotent operations.** Mutations carry idempotency keys (PRD §13.1); anything without one is not retried.

While capabilities live in one deployable (ADR-0001), cross-capability calls go through the interface the generated client will implement. **Do not stand up HTTP endpoints between modules that share a JVM** — that is latency and failure surface bought for nothing.

## Consequences

- A contract change that breaks a consumer breaks the build, which is the point.
- Generated code must be excluded from coverage metrics and from manual editing (`CLAUDE.md` §11).
- Resilience configuration is inert in Phase 1 and becomes load-bearing at extraction. It must be exercised by the resilience tests (PRD §20.1) before it is relied upon.

## Revisit when

A capability is extracted to its own deployable, or a genuinely asynchronous path appears (then see ADR-0013).
