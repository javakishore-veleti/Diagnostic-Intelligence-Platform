# Decision Queue

Unresolved material decisions. Nothing here is buried in chat output or code comments (`CLAUDE.md` §7.3).

**Blocking** means an agent must stop rather than assume. All fifteen PRD §27 decisions are blocking by definition until their ADR is accepted.

## Awaiting owner decision

| ID | Decision | ADR | Blocking | Recommendation |
|---|---|---|---|---|
| D-001 | Initial deployable count and module boundaries | [ADR-0001](../ADR/ADR-0001-initial-deployable-boundaries.md) | Yes | Modular monolith: 1 Java deployable, boundaries enforced by ArchUnit and per-schema DB users |
| D-002 | Maven vs Gradle | [ADR-0002](../ADR/ADR-0002-build-tool-and-orchestration.md) | Yes | Maven, for predictability under automation; `make verify` as the single green/red signal |
| D-003 | Angular workspace structure | [ADR-0003](../ADR/ADR-0003-angular-workspace-structure.md) | Yes | One Angular CLI workspace, 2 apps + 1 shared library, no Nx |
| D-004 | API client and resilience library | [ADR-0004](../ADR/ADR-0004-api-client-and-resilience.md) | Yes | Generated clients from OpenAPI + Resilience4j; no HTTP hops inside one JVM |
| D-005 | Identity realm, roles, synthetic users | [ADR-0005](../ADR/ADR-0005-identity-realm-and-roles.md) | Yes | Versioned realm export, 6 roles, `customer_id` claim enforced at the owning service |
| D-006 | Identifiers and state models | [ADR-0006](../ADR/ADR-0006-identifiers-and-state-models.md) | Yes | UUIDv7 keys with prefixed opaque API IDs; accession numbers as separate business fields; delay derived, never stored |
| D-007 | Synthea mapping subset | [ADR-0007](../ADR/ADR-0007-synthea-mapping-subset.md) | Yes | Import `Patient` and lab `Observation` only; manifest committed, bundles not |
| D-008 | LOINC licensing and subset | [ADR-0008](../ADR/ADR-0008-loinc-licensing-and-subset.md) | Yes | **Legal review required** — do not commit LOINC; curated mapping file only |
| D-009 | Embeddings, chunking, retrieval | [ADR-0009](../ADR/ADR-0009-embeddings-and-retrieval.md) | Phase 2 | `bge-small-en-v1.5`, 384 dims, CPU, heading-aware chunking |
| D-010 | Orchestration framework vs custom | [ADR-0010](../ADR/ADR-0010-assistant-orchestration.md) | Phase 2 | Thin custom Python loop; the execution record is the product |
| D-011 | Generative model and GPU | [ADR-0011](../ADR/ADR-0011-generative-model-and-gpu.md) | Phase 3 | Ollama + Qwen2.5-7B locally at zero cost; vLLM only for the bounded benchmark |
| D-012 | Conversation retention and audit | [ADR-0012](../ADR/ADR-0012-conversation-retention-and-audit.md) | Phase 3 | 30-day conversations, indefinite immutable execution records |
| D-013 | Kafka and Redis criteria | [ADR-0013](../ADR/ADR-0013-kafka-redis-criteria.md) | Yes | Neither in baseline; explicit entry criteria; outbox pattern from Phase 1 |
| D-014 | AWS hosting pattern | [ADR-0014](../ADR/ADR-0014-aws-hosting-pattern.md) | Phase 4 | Deferred. **Owner must supply a budget ceiling** and say whether Kubernetes is itself a goal |
| D-015 | Evaluation promotion thresholds | [ADR-0015](../ADR/ADR-0015-evaluation-promotion-thresholds.md) | Phase 2 | Absolute safety gates now; quality gates relative to a Phase 2 baseline |

## Owner input needed beyond approval

| ID | Question | Why it matters |
|---|---|---|
| D-008a | Do current LOINC licence terms permit the curated mapping extract in a public repository? | Legal. Should not rest on an agent's reading of the licence |
| D-014a | Monthly AWS budget ceiling | No honest hosting recommendation is possible without it |
| D-014b | Is Kubernetes experience itself a goal? | The only argument for EKS over ECS here |
| D-016 | The first 20–50 synthetic tests and panels (PRD §29) | Phase 1 catalog cannot be generated without it |

## Decided

_None yet._
