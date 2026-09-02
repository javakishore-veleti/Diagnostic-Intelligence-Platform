# ADR-0002: Build tool and monorepo orchestration

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.2
- **Requirements:** PRD §15 technology baseline, §21.1 path-aware CI

## Context

One build convention applies across all Java services. The monorepo also contains an Angular workspace (npm) and, from Phase 2, Python services — so a top-level orchestration story is needed regardless of the Java choice.

An additional constraint applies here that would not apply to a hand-written project: **most builds will be invoked by an agent, unattended.** The build tool's failure modes matter as much as its speed.

## Options

| | Maven | Gradle |
|---|---|---|
| Incremental build speed | Slower | Faster, with a build cache |
| Configuration | Declarative XML, one obvious way | Groovy/Kotlin scripts — real code |
| Failure modes | Verbose but deterministic | Faster, but script drift and cache staleness produce confusing failures |
| Agent editing | Structured, hard to corrupt subtly | A script an agent can make clever and wrong |
| Spring Boot docs | Primary | Equally supported |

## Decision

**Maven, multi-module, via the Maven Wrapper (`./mvnw`)**, with versions pinned in a root `dependencyManagement` block.

The deciding factor is not speed but predictability under automation. A declarative POM has one obvious shape; an agent editing it either produces valid XML or fails loudly. Gradle build scripts invite logic, and logic drifts — the resulting failures are slow to diagnose precisely when no human is watching. Gradle's speed advantage is real but matters most at a scale this project will not reach for a long time.

Orchestration across languages uses a root **`Makefile`** with a fixed vocabulary, so verification commands stay stable no matter what sits underneath:

```
make build      make test       make lint
make verify     # everything CI runs, the definition of green
make up / down / status / logs / reset   # delegate to DevOps/Local
```

`make verify` is the single command an unattended run treats as the pass/fail signal.

## Consequences

- Verification commands are stable across languages and survive tooling changes.
- Slower cold builds than Gradle; mitigated by module-scoped runs (`./mvnw -pl <module> test`) during the inner loop.
- CI path-aware workflows (PRD §21.1) map to module selection rather than Gradle's affected-project detection, which must be written explicitly.

## Revisit when

Cold build time passes roughly ten minutes, or a genuine need for build-level caching across CI runs appears.
