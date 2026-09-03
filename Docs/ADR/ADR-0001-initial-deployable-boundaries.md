# ADR-0001: Initial deployable count and module boundaries

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.1
- **Requirements:** PRD §9 capability map, §5.2 incremental deployables

## Context

The capability map names ten business capabilities. Building ten deployable Spring Boot applications on day one would be indefensible: each JVM costs roughly 0.5–1 GB of RAM, the local stack must run on one laptop alongside PostgreSQL, Keycloak, and two Angular dev servers, and there is no scaling requirement that justifies the split. PRD §9.2 explicitly permits combining capabilities into fewer deployables provided module boundaries and data ownership are preserved.

The risk of combining is that "logical boundaries" decay into a mud ball, and the extraction that was supposed to be possible later never is.

## Options

| Option | Deployables (Phase 1) | Assessment |
|---|---|---|
| One per capability | 10 Java + 2 Angular | Unrunnable on a laptop, no requirement justifies it |
| Experience / laboratory split | 3 Java + 2 Angular | Defensible, but the HTTP hop between them buys nothing while both are single-instance |
| **Modular monolith** | **1 Java + 1 Angular workspace** | Boundaries enforced by tests rather than by network; extraction stays cheap |

## Decision

Phase 1 ships **one Java deployable** (`platform`) containing every laboratory and experience capability as a separate build module, and **one Angular workspace** serving two applications.

Boundaries are enforced mechanically, not by convention:

- One build module per capability, with dependencies declared explicitly. A module may not depend on another capability's internals.
- One database schema per capability (PRD §12.5), each with its own migration path and its own database user. Cross-schema reads fail at the database level, not in review.
- **ArchUnit tests** in CI assert that no capability package imports another capability's internal package, and that experience modules never touch a laboratory repository directly.
- Cross-capability calls go through an interface that mirrors the eventual HTTP contract, so extraction is a transport change rather than a redesign.

Python intelligence services arrive in Phase 2 as a **second deployable**, because they are a different runtime, not because the boundary is more important.

## Consequences

- Local development runs two processes plus containers. Feasible on a laptop, and fast enough for an agent-driven test loop.
- A capability is extracted by giving its module a `main`, pointing callers at the generated HTTP client (ADR-0004), and moving its schema credentials. No domain rewrite.
- If the ArchUnit rules are ever deleted or suppressed to make a build pass, this decision has silently failed. Treat a suppression as an architecture change requiring an ADR.

## Revisit when

A capability needs independent scaling or deployment cadence, a second team owns one, or the single deployable's test suite becomes slow enough to discourage running it.
