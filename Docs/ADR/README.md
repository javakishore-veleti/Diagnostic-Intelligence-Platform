# Architecture Decision Records

Each ADR records one material decision, the options considered, and the consequences. ADRs are referenced by the PRD requirements they satisfy and are the authority for technical choices (PRD §31, `CLAUDE.md` §2).

**Status values:** `Proposed` (drafted, awaiting owner acceptance) → `Accepted` → `Superseded by ADR-NNNN`. No code may depend on a `Proposed` ADR.

## Milestone 0 — decisions required before scaffolding

These fifteen correspond one-to-one with the decisions PRD §27 forbids an agent from making silently.

| ADR | Decision | PRD §27 | Status |
|---|---|---|---|
| [0001](ADR-0001-initial-deployable-boundaries.md) | Initial deployable count and module boundaries | 1 | Proposed |
| [0002](ADR-0002-build-tool-and-orchestration.md) | Maven vs Gradle, monorepo build orchestration | 2 | Proposed |
| [0003](ADR-0003-angular-workspace-structure.md) | Angular workspace and shared UI library | 3 | Proposed |
| [0004](ADR-0004-api-client-and-resilience.md) | Synchronous API client and resilience library | 4 | Proposed |
| [0005](ADR-0005-identity-realm-and-roles.md) | Identity realm, roles, synthetic user mapping | 5 | Proposed |
| [0006](ADR-0006-identifiers-and-state-models.md) | Domain identifiers and state-transition models | 6 | Proposed |
| [0007](ADR-0007-synthea-mapping-subset.md) | Synthea-to-customer/result mapping subset | 7 | Proposed |
| [0008](ADR-0008-loinc-licensing-and-subset.md) | LOINC licensing, distribution, imported subset | 8 | Proposed |
| [0009](ADR-0009-embeddings-and-retrieval.md) | Embedding model, dimensions, chunking, retrieval | 9 | Proposed |
| [0010](ADR-0010-assistant-orchestration.md) | Orchestration framework vs thin custom | 10 | Proposed |
| [0011](ADR-0011-generative-model-and-gpu.md) | Initial generative model and GPU requirements | 11 | Proposed |
| [0012](ADR-0012-conversation-retention-and-audit.md) | Conversation retention and audit detail | 12 | Proposed |
| [0013](ADR-0013-kafka-redis-criteria.md) | Kafka and Redis introduction criteria | 13 | Proposed |
| [0014](ADR-0014-aws-hosting-pattern.md) | AWS container and model-runtime hosting | 14 | Proposed |
| [0015](ADR-0015-evaluation-promotion-thresholds.md) | Evaluation thresholds for promotion | 15 | Proposed |

## Template

```markdown
# ADR-NNNN: <decision>

- **Status:** Proposed | Accepted | Superseded by ADR-NNNN
- **Date:** YYYY-MM-DD
- **PRD decision:** §27.N
- **Requirements:** <FR-/NFR- IDs>

## Context
## Options
## Decision
## Consequences
## Revisit when
```
