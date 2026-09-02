# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Claude Code Long-Running Agent Operating Guide

## Diagnostic Intelligence Platform

**Purpose:** Repository-level instructions and persistent project memory for the Claude Code CLI.
**Applies to:** Every file and directory in this repository unless a more specific nested `CLAUDE.md` adds compatible local instructions.
**Product source of truth:** `PRD.md`
**Architecture source of truth:** `REQUIREMENTS-ARCHITECTURE.md`, accepted ADRs, and versioned API/event contracts

---

## 0. Current Repository State

As of the last update to this file, the repository contains **no application code**. Present files: `PRD.md`, `README.md`, `CLAUDE.md`, `LICENSE`, `.gitignore`. Nothing in Sections 1–26 describes existing code; it describes the contract that new code must satisfy.

None of the supporting documents referenced throughout this guide exist yet — `REQUIREMENTS-ARCHITECTURE.md`, `LOCAL-DEVELOPMENT.md`, `SECURITY.md`, `CONTRIBUTING.md`, `Docs/ADR/`, `Docs/Execution/`, the `DevOps/Local/*.sh` scripts, and all OpenAPI/event contracts. Create them as their milestone requires; do not treat their absence as permission to skip the rule that depends on them, and do not create them as empty placeholders.

Verify with `ls`/`git status` before assuming any path exists. Never assume the target architecture has already been implemented merely because it appears in the PRD tree.

---

## 1. Mission

Build the Diagnostic Intelligence Platform incrementally as a production-style portfolio project. The platform helps synthetic healthcare members access laboratory services and enables operations teams to manage orders, specimens, results, exceptions, knowledge, and service insights.

The repository is a single GitHub monorepo with these business-oriented areas:

- `Portals/DiagnosticsAdmin`
- `Portals/DiagnosticServices`
- `Middleware/ExperienceServices`
- `Middleware/LaboratoryServices`
- `Middleware/DiagnosticIntelligence`
- `DataMgmt`
- `DevOps`
- `Testing`
- `Docs`

Claude must optimize for a verified, maintainable sequence of vertical slices—not maximum code volume.

---

## 2. Authority and Precedence

When instructions conflict, use this order:

1. The human user's current explicit instruction.
2. Security, privacy, legal, and safety requirements.
3. `PRD.md` product requirements and non-goals.
4. Accepted Architecture Decision Records in `Docs/ADR`.
5. `REQUIREMENTS-ARCHITECTURE.md`.
6. Versioned API, event, and data contracts.
7. This root `CLAUDE.md`.
8. A more specific nested `CLAUDE.md` for implementation details within its directory, provided it does not contradict higher-level sources.
9. Existing code conventions and tool defaults.

Do not silently resolve material contradictions. Record the conflict in the active plan and ask the owner when the decision changes scope, architecture, security, data ownership, cost, or public behavior.

---

## 3. Mandatory First Actions

At the beginning of every new session or after losing context:

1. Read this file completely.
2. Read `PRD.md` completely.
3. Read `REQUIREMENTS-ARCHITECTURE.md` if it exists.
4. Read `README.md`, `LOCAL-DEVELOPMENT.md`, and `SECURITY.md` if they exist.
5. Read the current execution state files listed in Section 7.
6. Read all accepted ADRs relevant to the intended change.
7. Inspect the actual repository and git status before proposing work.
8. Identify the relevant PRD requirement IDs.
9. Run the smallest relevant baseline verification before editing.
10. Present or update a bounded implementation plan.

Never assume the target architecture has already been implemented merely because it appears in the PRD tree.

### 3.1 Requirement identifiers

Requirement IDs are stable and referenced in issues, tests, and pull requests. They take the form `FR-<CAP>-NNN` and `NFR-<AREA>-NNN` — for example `FR-AST-004` (assistant), `FR-ADM-011` (admin portal), `FR-CAT-003` (catalog), `FR-CUS-002` (customer), `NFR-PERF-001`. Never reuse an ID for a different meaning; deprecated requirements stay documented with a replacement reference. Material changes to scope, safety boundaries, capability ownership, data sources, or acceptance criteria require a PRD version update (PRD §31). Generated code is subordinate to the accepted PRD, contracts, migrations, and ADRs.

---

## 4. Non-Negotiable Product Boundaries

Agents must preserve these rules:

- All people, orders, specimens, results, and operational records are synthetic.
- Do not add real patient data, protected health information, employer data, production logs, or proprietary documents.
- Do not use Quest Diagnostics branding, logos, proprietary identifiers, internal architecture, or copied content.
- Do not represent the platform as a real clinical product.
- Do not diagnose disease, recommend treatment, prescribe medication, or replace a clinician.
- An LLM is never the system of record.
- Authoritative order, specimen, result, identity, authorization, and timestamp facts come from deterministic services.
- LLM-accessible tools are allowlisted, schema-validated, bounded, and read-only initially.
- The model may not directly change orders, specimens, results, catalogs, permissions, knowledge publication state, or audit records.
- Missing or conflicting evidence must result in an explicit limitation, not an invented answer.
- Customer data isolation must be enforced by services, not by the prompt or portal alone.

Stop immediately and report the issue if requested work would violate one of these boundaries.

Additional PRD non-goals: no billing, insurance adjudication, claims processing, appointment scheduling, provider ordering, or real notifications in the initial release; no claim of HIPAA compliance, CLIA certification, or FDA clearance. Every portal must display a notice that all people, orders, specimens, and results are synthetic and the product is for demonstration and educational use only.

### 4.1 Approved and prohibited data sources (PRD §12)

Approved: Synthea FHIR R4 (pin generator/dataset version, record provenance); LOINC (follow license, record release version, import only required subsets); project-owned `SyntheticDataGen` output; a project-authored synthetic test catalog using project codes such as `LAB-10001`, mapped to LOINC; project-authored synthetic SOPs labeled synthetic/educational; approved public references with source and terms metadata (no bulk scraping); CMS synthetic claims datasets only as separate future scope.

Prohibited: real patient or employee data; data retained from prior employment; proprietary laboratory documents or screenshots; credentials, tokens, production logs, or internal API responses; unlicensed bulk copies of commercial content; public content whose terms prohibit the intended use.

---

## 5. Architecture Guardrails

### 5.1 Business naming

Use business-capability names at repository and service boundaries. Keep implementation terms below those boundaries.

Good:

```text
Middleware/DiagnosticIntelligence/KnowledgeServices/retrieval
Middleware/DiagnosticIntelligence/AssistantServices/model-access/vllm
```

Avoid:

```text
Middleware/RagService
Middleware/VllmService
Portals/CustomerPortal
```

Canonical business terms (PRD §4.3):

| Term | Meaning |
|---|---|
| DiagnosticServices | Customer-facing digital experience and its experience API |
| DiagnosticsAdmin | Internal operations and administration experience and its experience API |
| LaboratoryServices | Business services for customers, orders, specimens, tests, and results |
| DiagnosticIntelligence | Assistant, knowledge, insight, and intelligence-quality capabilities |
| DataMgmt | Synthetic generation, ingestion, reference data, schemas, and knowledge data |
| Customer | A synthetic person using DiagnosticServices, represented with FHIR resources where applicable |
| Laboratory order | A request for one or more laboratory tests |
| Specimen | Collected material associated with an order and one or more tests |
| Result | A synthetic laboratory observation or panel result |
| Operational event | An immutable business event describing progress or an exception |
| Knowledge source | An approved document used for retrieval and grounded answers |

The product and repository name is `diagnostic-intelligence-platform`; it must never reference Quest, a past employer, or another real laboratory company.

### 5.2 Incremental deployables

The final capability map does not require one deployable per folder on day one.

- Prefer a small number of modular applications during early milestones.
- Preserve module and schema ownership so capabilities can be extracted later.
- Do not generate empty microservices merely to match the target tree.
- A planned but unimplemented capability may have a concise boundary README instead of scaffolded code.
- Creating or splitting a deployable requires an accepted ADR.

### 5.3 Experience API boundary

- Portals call their corresponding experience services.
- Portals do not call LaboratoryServices or DiagnosticIntelligence services directly.
- Experience services compose responses but do not own laboratory systems-of-record data.

Capability ownership (PRD §9):

- `ExperienceServices/DiagnosticServicesApi` serves only the DiagnosticServices portal; `ExperienceServices/DiagnosticsAdminServices` serves only the DiagnosticsAdmin portal. Both compose data, enforce experience-specific authorization, shape responses, and expose streaming endpoints.
- `LaboratoryServices` owns the record: `CustomerServices` (synthetic identity/profile projections), `LabOrderServices` (orders, ordered tests, order state and events), `SpecimenServices` (specimens, requirements, accession-like identifiers, state and events), `TestCatalogServices` (test definitions and terminology mappings), `ResultServices` (results, panels, values, units, ranges, flags, status, release state).
- `DiagnosticIntelligence` owns: `AssistantServices` (conversation orchestration, approved tool selection, streaming, safety policies, prompts, model access), `KnowledgeServices` (documents, versions, parsing, chunking, embeddings, retrieval, citation metadata), `InsightsServices` (deterministic operational insights — delayed specimens, turnaround trends, exception aggregation), `QualityServices` (evaluation definitions, runs, scores, comparisons, release recommendations).
- `SharedServices` must stay small and stable: cross-cutting security helpers, observability conventions, error envelopes, correlation identifiers, test fixtures, generated API clients. It must not become a dumping ground for domain models or shared database entities.

### 5.4 Data ownership

- Each capability owns its schema and migrations.
- A service must not query or update another capability's tables.
- Cross-capability communication uses versioned APIs, events, or deliberately owned read models.
- Do not share mutable ORM/domain entities across services.
- Shared libraries are restricted to stable cross-cutting contracts and utilities.

Local environments may run one PostgreSQL instance, but logical ownership is mandatory (PRD §12.5):

| Schema | Owner |
|---|---|
| `customer` | CustomerServices |
| `catalog` | TestCatalogServices |
| `lab_order` | LabOrderServices |
| `specimen` | SpecimenServices |
| `result` | ResultServices |
| `knowledge` | KnowledgeServices |
| `assistant` | AssistantServices |
| `quality` | QualityServices |
| `identity` | Local identity provider or isolated identity integration |

### 5.5 Technology restraint

Do not add any of the following without an accepted requirement and ADR:

- Kafka
- Redis
- Kubernetes
- A separate vector database
- A second API gateway
- GraphQL
- A service mesh
- A workflow engine
- A second model orchestration framework
- A new cloud provider
- One database instance per service in local development

PostgreSQL plus pgvector is the initial storage baseline. vLLM is accessed through an internal OpenAI-compatible model-access abstraction.

### 5.6 Technology baseline (PRD §15)

Versions are pinned at project initialization using currently supported stable/LTS releases and recorded in an ADR; the PRD deliberately does not guess versions.

| Area | Baseline |
|---|---|
| Java services | Java 21 or newer supported LTS; Spring Boot 3.x |
| Build | Maven or Gradle, selected once in an ADR, one convention across Java services |
| Web portals | Current supported Angular/TypeScript with workspace and library conventions |
| AI/knowledge services | Supported Python 3.x with reproducible dependency locking |
| Primary database | PostgreSQL |
| Vector search | pgvector inside PostgreSQL initially |
| Cache | Redis only when Phase 3 use cases justify it |
| Messaging | Kafka only when event use cases justify it |
| Identity | Keycloak locally; cloud identity remains replaceable |
| Model serving | vLLM exposing an OpenAI-compatible endpoint |
| Embeddings | Replaceable provider/model with a pinned version |
| API contract | OpenAPI 3.1 |
| Migrations | Flyway for Java-owned schemas; a documented equivalent elsewhere |
| Observability | OpenTelemetry, Prometheus, Grafana, structured logs; Loki optional |
| Containers | Docker and Docker Compose |
| CI/CD | GitHub Actions, manual deployment initiation initially |
| Infrastructure | Terraform |
| AWS compute | ECS/EKS choice for services and GPU hosting decided by ADR |
| Testing | JUnit/Testcontainers, Angular tests, Python tests; Playwright/Cypress and k6/Gatling chosen by ADR |

Model portability is a requirement, not a preference: services call an internal model-access interface using an OpenAI-compatible contract, so vLLM can be replaced by OpenAI, Azure OpenAI, or Bedrock through an adapter without changing portal APIs or laboratory domain logic.

### 5.7 Delivery phasing (PRD §8)

- **Phase 1 (MVP)** delivers a useful system *without* generative AI: monorepo foundation, both Angular shells, both experience APIs, catalog/order/specimen capabilities, PostgreSQL with separated schemas, synthetic data, search and detail views, deterministic timeline and delay calculation, local Docker scripts, OpenAPI contracts, migrations, tests, observability baseline, CI.
- **Phase 2** adds customer and result capabilities, Synthea ingestion, LOINC reference data, curated synthetic SOPs, document lifecycle through pgvector retrieval and citations, a knowledge-only grounded assistant, and the AI evaluation baseline.
- **Phase 3** adds read-only tool calling, evidence-backed delay investigation, streaming with ordered events, conversation persistence, justified Redis use, justified Kafka use, and end-to-end OpenTelemetry traces.
- **Phase 4** adds Terraform-managed AWS, full GitHub Actions delivery, managed PostgreSQL, GPU-backed vLLM or provider fallback, and evaluation gates.

Do not pull later-phase infrastructure forward into an earlier milestone.

---

## 6. Long-Running Agent Work Model

Every substantial change is executed as a bounded work package. A work package should normally fit one coherent pull request and leave the repository runnable.

### 6.1 Work-package lifecycle

1. **Orient** — reload product, architecture, repository, git, and execution state.
2. **Define** — state the outcome, PRD IDs, scope, non-scope, dependencies, risks, and verification.
3. **Plan** — break the work into independently verifiable tasks.
4. **Baseline** — run relevant existing tests/builds before modifying code.
5. **Implement** — make the smallest coherent change.
6. **Verify continuously** — run focused tests after each meaningful step.
7. **Integrate** — run broader affected-component checks.
8. **Review** — inspect the diff for scope, security, data boundaries, and accidental files.
9. **Checkpoint** — update durable state and create a git commit only if authorized.
10. **Handoff** — report completed work, evidence, remaining risks, and the next recommended work package.

### 6.2 One active objective

Maintain one active work-package objective. Do not opportunistically implement unrelated features. Record useful discoveries in the backlog instead of expanding the active scope.

### 6.3 Vertical slices

Prefer a complete thin slice such as:

```text
Synthetic catalog seed
  → catalog persistence
  → versioned API
  → experience API projection
  → portal read-only view
  → tests and telemetry
```

Avoid broad horizontal generation such as “create all entities,” “create all services,” or “create every portal page” without an end-to-end verified capability.

### 6.4 Time and context limits

Long-running does not mean unbounded.

- Reassess the plan after every major task.
- Create a state checkpoint before context becomes uncertain.
- Never continue from memory when durable state can be read.
- If a command appears hung, investigate; do not wait indefinitely.
- If the same failure recurs three times without new evidence, stop and document the blocker.
- If the change grows materially beyond the approved work package, stop and propose a follow-up package.

### 6.5 Claude Code session protocol

- Begin architecture or multi-file work in Plan Mode when available.
- Do not leave Plan Mode and edit until the owner approves the work package.
- Use Claude Code's task list for immediate actions, but treat `Docs/Execution/CURRENT-WORK.md` as the durable source that survives compaction and new sessions.
- Before `/compact`, update `CURRENT-WORK.md` with the last verified state and next exact command/action.
- After `/compact`, `/resume`, or a new CLI session, reread the mandatory files and reconcile them with `git status` before continuing.
- Do not rely on auto-memory or prior conversation summaries for product or architecture decisions that belong in repository documentation.
- Ask before using `--dangerously-skip-permissions`; the normal project workflow must not require it.
- Prefer narrowly scoped permissions in `.claude/settings.json` when the owner chooses to add them. Never broaden allowed commands merely to avoid a prompt.
- Do not modify user-level Claude configuration, global memory, MCP configuration, plugins, hooks, or credentials unless explicitly requested.
- Do not launch unbounded background processes. If a development server is required, record how it was started, how readiness is checked, and how it is stopped.

### 6.6 Planning versus execution sessions

For large milestones, use separate Claude Code sessions:

1. **Planning session:** inspect only, identify requirements and ADRs, create the work-package proposal, and wait for approval.
2. **Execution session:** implement only the approved package and maintain durable state.
3. **Review session:** inspect the complete diff, run broad verification, and report gaps without quietly adding new scope.

The same session may perform all three only when context remains reliable and each transition is explicit.

---

## 7. Durable Execution State

Long-running agents must not depend on chat history. Maintain these files when implementation begins:

```text
Docs/Execution/
├── CURRENT-WORK.md
├── IMPLEMENTATION-ROADMAP.md
├── DECISION-QUEUE.md
├── RISK-REGISTER.md
└── SESSION-LOG.md
```

Do not create them as empty placeholders. Create them when Milestone 0 starts, using the structures below.

### 7.1 `CURRENT-WORK.md`

This is the authoritative resumable state for the current work package.

```markdown
# Current Work

## Work package
- ID:
- Title:
- Status: PLANNED | IN_PROGRESS | BLOCKED | VERIFYING | COMPLETE
- Owner/agent:
- Started:
- Last updated:

## Product linkage
- PRD requirement IDs:
- Milestone:
- Relevant ADRs:

## Intended outcome

## In scope

## Out of scope

## Plan
- [ ] Task with verification

## Current checkpoint
- Last completed task:
- Current branch/commit:
- Working tree state:
- Next exact action:

## Verification evidence
| Command | Result | Time |
|---|---|---|

## Decisions and assumptions

## Blockers

## Files changed
```

Update this file after every major verified step, before a session ends, and before handing work to another agent.

### 7.2 `IMPLEMENTATION-ROADMAP.md`

Track approved work packages, not speculative code tasks.

Each entry includes:

- Work-package ID and title.
- PRD requirement IDs.
- Dependency packages.
- Deliverable and acceptance test.
- Status.
- Link to issue/PR/commit when available.

Only one work package may be marked `IN_PROGRESS` unless the owner explicitly authorizes parallel work with non-overlapping file ownership.

### 7.3 `DECISION-QUEUE.md`

Record unresolved material decisions with:

- Decision ID.
- Question.
- Why it matters.
- Options and trade-offs.
- Recommended option.
- Blocking or non-blocking status.
- Owner decision and date.
- ADR to create/update.

Do not bury material decisions in chat output or code comments.

### 7.4 `RISK-REGISTER.md`

Track active risks with likelihood, impact, mitigation, owner, trigger, and status. Include security, safety, privacy, architecture, licensing, cost, and delivery risks.

### 7.5 `SESSION-LOG.md`

Append concise session entries only when they provide recovery value:

- Date/time.
- Agent/tool identity if useful.
- Work-package ID.
- Completed outcomes.
- Verification summary.
- Blocker or handoff.

Do not paste raw command output, chain-of-thought, secrets, or lengthy narratives.

---

## 8. Planning Requirements

Before editing code for a substantial work package, document:

- Intended user/business outcome.
- Applicable PRD requirement IDs.
- Acceptance criteria.
- Components and files expected to change.
- API, event, schema, or UI contract changes.
- Security and authorization effects.
- Data migration or seed-data effects.
- Observability requirements.
- Testing strategy.
- Explicit non-scope.
- Rollback or recovery strategy.
- Open decisions.

If the architecture or version is not selected, create an ADR proposal instead of silently selecting a tool.

### 8.1 Work-package sizing

Split a package when it:

- Spans unrelated business capabilities.
- Requires multiple architectural decisions.
- Cannot be verified with a clear end state.
- Mixes infrastructure migration with a large feature.
- Would introduce multiple new runtime technologies.
- Requires a long-running agent to modify overlapping areas without checkpoints.

---

## 9. Multi-Agent and Parallel Work

Use Claude Code subagents or agent teams only when the owner explicitly authorizes parallel agent work.

When parallel work is authorized:

- Assign non-overlapping work packages and file ownership.
- Designate one coordinating agent.
- Record assignments in `CURRENT-WORK.md` or a linked work-package file.
- Do not allow two agents to edit the same migration, contract, shared configuration, or generated lockfile concurrently.
- Freeze shared contracts before parallel consumers/providers implement against them.
- Each agent verifies its own work and returns changed files, commands, results, decisions, and risks.
- The coordinator integrates sequentially and runs repository-level verification.

Claude-specific constraints:

- Give each subagent a concrete read-only investigation or non-overlapping implementation task.
- Do not delegate product decisions or final architecture authority.
- Require every subagent to return files inspected/changed, verification executed, assumptions, and blockers.
- Do not let subagents recursively spawn more agents unless explicitly authorized.
- Do not use subagents as a substitute for reading `PRD.md`, `CLAUDE.md`, relevant ADRs, and contracts in the coordinating context.

Do not use parallel agents for tightly coupled tasks where communication cost exceeds execution benefit.

---

## 10. Git and Change Management

### 10.1 Before changes

- Inspect `git status`, current branch, recent commits, and relevant diffs.
- Treat uncommitted changes as user-owned unless proven otherwise.
- Do not discard, overwrite, reformat, or include unrelated changes.
- Identify generated files and repository-specific commands.

### 10.2 Branches and commits

- Do not create branches, commits, tags, pushes, or pull requests unless authorized by the owner or established workflow.
- When commits are authorized, keep them coherent and verifiable.
- Commit messages should describe the business change, not merely the files.
- Do not rewrite published history or force-push without explicit authorization.
- Do not bypass required checks.

Suggested branch pattern:

```text
feature/<work-package-id>-<short-business-name>
fix/<work-package-id>-<short-business-name>
docs/<work-package-id>-<short-name>
```

### 10.3 Diff discipline

Before handoff:

- Review the full diff.
- Remove debugging code and accidental files.
- Confirm no secrets, real data, model weights, generated datasets, local environment files, or bulky reports are included.
- Confirm formatting changes are scoped.
- Confirm requirement and documentation updates accompany behavior changes.

---

## 11. Code Generation Rules

Agents may generate code only within the approved work package.

- Do not scaffold all planned portals and services in one operation.
- Do not create speculative abstractions for possible future requirements.
- Do not duplicate models and utilities without checking existing modules.
- Prefer framework-standard patterns over bespoke infrastructure.
- Pin dependency versions through the established dependency-management mechanism.
- Do not use `latest` container tags in reproducible environments.
- Generated API clients must derive from versioned contracts.
- Generated code must be reproducible and clearly separated from handwritten code.
- Never manually edit generated files unless the repository explicitly treats them as source.
- Avoid TODOs that hide incomplete safety, authorization, persistence, or error handling.

If the requested output is only planning or documentation, do not generate application code.

---

## 12. API and Contract Discipline

- Define or update OpenAPI/event schemas before or with implementation.
- Use the repository's standard RFC 9457-based error envelope.
- Use opaque IDs and UTC ISO 8601 timestamps.
- Define pagination, filtering, sorting, idempotency, and maximum sizes explicitly.
- Preserve backward compatibility unless a versioned breaking change is approved.
- Add provider and consumer contract tests.
- Do not expose persistence entities directly as API payloads.
- Do not leak vLLM, pgvector, Kafka, database, or internal service details into portal contracts.
- Every request path must propagate correlation and trace context.

For streaming:

- Use the PRD-defined Server-Sent Events contract.
- Emit monotonically increasing sequence numbers.
- Preserve declared output order even when safe operations run concurrently.
- Never stream hidden prompts, chain-of-thought, secrets, or raw stack traces.

### 12.1 Concrete conventions (PRD §13.1)

- REST/JSON is the default synchronous integration style; OpenAPI 3.1 specifications are source-controlled.
- Versioned base path such as `/api/v1`.
- IDs are opaque strings/UUIDs, never sequential database keys.
- Timestamps are ISO 8601 UTC.
- List endpoints provide pagination, filtering, stable sorting, and documented limits.
- Mutation endpoints use idempotency keys wherever replay is possible.
- Errors use one consistent RFC 9457 problem-details contract.
- Every response carries or returns a correlation/request identifier.
- Generated clients are allowed, but contracts remain capability-owned.

Exact paths and payload schemas live in capability OpenAPI documents. Do not invent multiple overlapping endpoints without updating those contracts.

Planned experience surfaces — `DiagnosticServicesApi`: customer profile summary, order list and detail, specimen timeline for an authorized order, released result list and detail, assistant conversation creation and SSE streaming. `DiagnosticsAdminServices`: operations dashboard, cross-capability search, investigation read model, catalog administration, knowledge administration, synthetic data job administration, quality run administration and results, platform health summaries.

### 12.2 Events

Event-driven integration is added only when a use case requires decoupling, replay, fan-out, or asynchronous processing. Candidate events: `LaboratoryOrderSubmitted`, `SpecimenCollected`, `SpecimenReceived`, `SpecimenProcessingDelayed`, `SpecimenRejected`, `ResultFinalized`, `ResultCorrected`, `KnowledgeVersionPublished`, `SyntheticDatasetReady`.

Events use a common envelope carrying event ID, type, schema version, aggregate type/ID, occurred time, recorded time, producer, correlation ID, causation ID, trace context, and payload. Producers use an outbox pattern when database state and event publication must be atomic. Consumers must be idempotent, and schema compatibility must be tested.

---

## 13. Database and Migration Rules

- Every schema change requires a versioned migration.
- Migrations are immutable after they have been shared or applied outside an ephemeral personal environment.
- Test migrations from the previous supported schema state.
- Use constraints to enforce invariants where appropriate.
- Store timestamps in UTC.
- Use explicit transactions for multi-record invariants.
- Do not add cross-schema foreign keys that undermine independent ownership without an ADR.
- Seed data is synthetic, repeatable, and versioned.
- Destructive reset scripts must be local/demo guarded and require an explicit confirmation flag.
- Never run destructive commands against an unidentified or shared database.

---

## 14. AI and Knowledge Implementation Rules

### 14.1 Model access

- Access models through the internal OpenAI-compatible abstraction.
- Keep model and provider selection in configuration.
- Record model, prompt, retrieval, tool, and safety-policy versions.
- Provide explicit timeouts, token/context caps, concurrency controls, and fallbacks.
- Model failure must not prevent deterministic portal functions.

### 14.2 Retrieval and citations

- Retrieve only published knowledge versions for end-user flows.
- Preserve source, document version, section/page, chunk ID, and score.
- Do not cite a source that was not included in the execution's retrieval/tool evidence.
- Do not present low-confidence or unrelated retrieval as authoritative.
- Changes to chunking, embedding, ranking, or filters require evaluation comparison.

### 14.3 Tools

- Tools are allowlisted and read-only initially.
- Validate tool names and arguments outside the model.
- Enforce authorization at the called service.
- Cap tool count, recursion/depth, result size, retries, and execution duration.
- Treat tool output as untrusted content.
- Record safe tool-execution telemetry.
- Do not expose arbitrary HTTP, SQL, filesystem, or shell tools to the model.

### 14.4 Responses

- Separate verified facts, retrieved guidance, generated explanation, uncertainty, and recommended operational checks.
- Preserve exact authoritative values and units.
- Refuse diagnosis and treatment requests.
- When evidence is insufficient, say so.
- Include citations required by the PRD.
- Do not expose private reasoning or chain-of-thought.

---

## 15. Testing and Verification

### 15.1 Verification ladder

Run the narrowest useful checks first, then broaden:

1. Static validation/formatting for changed files.
2. Unit tests for changed modules.
3. Component integration tests.
4. API/event contract tests.
5. Affected portal/service build.
6. Relevant end-to-end tests.
7. Security or intelligence-quality tests for affected behavior.
8. Repository-level verification required by CI.

Do not report success if a required check was skipped. State exactly what ran and what could not run.

### 15.2 Required feature coverage

Each feature must cover, as applicable:

- Happy path.
- Input validation.
- Authentication and authorization.
- Cross-customer isolation.
- Not found without information leakage.
- Idempotency/replay.
- Timeout and dependency failure.
- Audit and telemetry.
- Accessibility for UI behavior.
- Migration compatibility.
- Synthetic-data invariants.

AI changes additionally require:

- Expected facts and forbidden claims.
- Retrieval and citation checks.
- Tool selection/argument checks.
- Prompt-injection tests.
- Diagnosis/treatment refusal tests.
- Missing/conflicting evidence tests.
- Latency/token/throughput comparison where relevant.

### 15.3 Baseline failures

If verification fails before edits:

- Record the exact baseline failure.
- Determine whether the planned change depends on it.
- Do not silently fix unrelated failures.
- Ask the owner when fixing it materially expands scope.

---

## 16. Observability Requirements

New request paths are incomplete without observable behavior.

- Emit structured logs without secrets or unnecessary synthetic customer/result detail.
- Add trace spans for service calls, database work, retrieval, tools, and model access.
- Emit rate, error, and duration metrics.
- Propagate request, correlation, trace, and causation identifiers.
- Include service name, version, and environment.
- Add health/readiness behavior for new dependencies.
- Document meaningful alerts or dashboards for operationally significant capabilities.

Do not log hidden prompts, credentials, tokens, full model context, or raw tool responses by default.

---

## 17. Security Review Checklist

Before completing a work package, verify:

- Authentication and role checks occur at the API boundary.
- Object-level authorization prevents cross-customer access.
- Inputs, uploads, filenames, content types, and sizes are validated.
- No secrets are committed or printed.
- Logs and traces minimize sensitive fields.
- Database accounts follow schema ownership.
- External calls use allowlisted destinations and timeouts.
- Dependencies and images are scanned.
- AI tools cannot invoke arbitrary operations.
- Retrieved/tool content cannot override system policy.
- New administrative actions are audited.
- Destructive operations have environment guards and confirmation.

If any applicable item is missing, the work package is not complete.

---

## 18. Local Development Rules

Preserve the developer experience defined by:

- `DevOps/Local/docker-all-up.sh`
- `DevOps/Local/docker-all-down.sh`
- `DevOps/Local/docker-all-status.sh`
- `DevOps/Local/docker-all-logs.sh`
- `DevOps/Local/docker-all-reset.sh`

Agents must:

- Keep scripts callable from any directory.
- Make up/down/status idempotent.
- Wait for readiness, not only container creation.
- Keep vLLM optional because of GPU requirements.
- Allow application services to run from IDEs while infrastructure runs in containers.
- Keep ports, profiles, prerequisites, and troubleshooting current in `LOCAL-DEVELOPMENT.md`.
- Avoid introducing a dependency into the `core` profile before its milestone.

### 18.1 Profiles and script behavior (PRD §16)

Profiles: `core` (PostgreSQL and Keycloak initially), `intelligence` (model runtime and required embedding components), `observability` (collector, metrics, dashboards, log tooling), `all`. Redis and Kafka stay out until their implementation phases.

Script contract:

- `docker-all-up.sh` starts the selected profile in dependency order and waits for readiness.
- `docker-all-down.sh` stops the stack without deleting data by default, idempotently.
- `docker-all-status.sh` reports component, container state, readiness, health URL, port, and dependency failures.
- `docker-all-logs.sh` displays or follows logs with optional component filtering.
- `docker-all-reset.sh` is explicitly destructive, guarded, local-only, and documented.

Scripts resolve the repository root safely so they work from any current directory. Secrets come from ignored local environment files or a secret manager, never committed defaults. Resource-intensive vLLM validates GPU/runtime prerequisites; a lightweight OpenAI-compatible stub or an explicitly configured remote provider supports development without a GPU.

---

## 19. Dependency and Upgrade Policy

- Use supported stable/LTS framework versions chosen through Milestone 0 ADRs.
- Pin direct dependencies and container images.
- Use dependency locks/BOMs/catalogs consistently.
- Do not combine broad dependency upgrades with unrelated feature work.
- Review release notes and migration guides for material upgrades.
- Run affected tests and record compatibility changes.
- Replace abandoned or vulnerable dependencies through a dedicated work package.

---

## 20. Documentation Requirements

Update documentation in the same work package when behavior changes.

Potentially affected artifacts include:

- `PRD.md` for product-level changes.
- `REQUIREMENTS-ARCHITECTURE.md` for architecture constraints and views.
- `Docs/ADR` for material technical decisions.
- OpenAPI/event contracts.
- Data model and state-transition documentation.
- `LOCAL-DEVELOPMENT.md` for developer workflow.
- `SECURITY.md` and threat models.
- Runbooks and observability dashboards.
- Execution-state files.

Documentation must describe the current implemented reality and clearly label future design.

### 20.1 Claude Code configuration files

Repository-specific Claude automation may be introduced only through a reviewed work package:

```text
.claude/
├── settings.json
├── commands/
└── agents/
```

- `settings.json` must contain only project-safe permissions and hooks.
- Custom commands must be deterministic wrappers around documented project workflows.
- Hooks must be fast, scoped, visible, and unable to expose secrets or destructively mutate unrelated files.
- Project subagent definitions must repeat their boundaries, allowed areas, expected output, and verification responsibility.
- Do not commit local-only settings or credentials.
- Do not create Claude configuration merely because the directory is supported; add it only when it improves a documented workflow.

---

## 21. Blocker and Escalation Rules

Stop and ask the owner when:

- A Section 27 decision from `PRD.md` is required and unresolved.
- Requirements conflict materially.
- A change crosses a safety, legal, licensing, security, privacy, or medical boundary.
- Real or proprietary data is encountered.
- Credentials or permissions are missing.
- A destructive action would affect an unclear target.
- The change requires a new paid/cloud resource or materially increases cost.
- The initial work package must expand across unrelated capabilities.
- An API, event, or database breaking change lacks approval.
- The working tree contains overlapping user changes that cannot be preserved safely.
- Verification cannot be performed and the unverified result would create meaningful risk.

When blocked, report:

1. The exact blocking fact.
2. Evidence already gathered.
3. Why safe progress cannot continue.
4. Two or three concrete options with trade-offs.
5. The recommended decision.
6. The exact next action after the decision.

### 21.1 Decisions the agent must never make silently (PRD §27)

Create or update an ADR and obtain owner acceptance for each of these before code depends on it:

1. Initial deployable-service count and module boundaries.
2. Maven versus Gradle and monorepo build orchestration.
3. Angular workspace and shared UI library structure.
4. Synchronous API client approach and resilience library.
5. Initial identity realm, roles, and synthetic user mapping.
6. Domain identifiers and detailed state-transition models.
7. Synthea-to-customer/result mapping subset.
8. LOINC licensing/distribution handling and imported subset.
9. Embedding model, vector dimensions, chunking strategy, and retrieval baseline.
10. Chat/orchestration framework versus deliberately thin custom orchestration.
11. Initial open-source generative model and local/AWS GPU requirements.
12. Conversation retention and audit detail.
13. Kafka and Redis introduction criteria.
14. AWS container and model-runtime hosting pattern.
15. Evaluation thresholds for promotion.

Open product decisions that remain with the owner (PRD §29) include the final public product name, whether `DiagnosticServices` stays the customer-facing brand, the exact Phase 1 deployable grouping, the first 20–50 synthetic tests/panels and workflow rules, the initial vLLM-served model, the AWS budget and ECS/EKS/EC2 preference, and how the demonstration is published.
---

## 22. Failure Recovery

When a command, build, migration, or test fails:

1. Capture the concise error and failing command in `CURRENT-WORK.md`.
2. Determine whether it existed at baseline.
3. Reproduce with the smallest command.
4. Form one evidence-based hypothesis.
5. Apply the smallest safe correction.
6. Re-run the narrow test, then the broader affected checks.
7. Revert only the agent's own unsuccessful change when necessary; never discard unrelated work.

If interrupted, the next agent must be able to resume from `CURRENT-WORK.md` without relying on chat history.

---

## 23. Checkpoint Protocol

Create a checkpoint after:

- Completing a migration or public contract.
- Completing a vertical-slice layer.
- Passing a meaningful verification stage.
- Making an accepted architecture decision.
- Before a risky or long-running operation.
- Before handing work to another agent.
- Before ending the session.

A checkpoint consists of:

- Updated `CURRENT-WORK.md`.
- Cleanly saved files.
- Verification commands and results.
- Current git status/commit reference.
- Known failures or risks.
- One exact next action.

A git commit is a checkpoint only when commit authorization exists. Otherwise, durable state plus the working-tree diff is the checkpoint.

---

## 24. Completion and Handoff Format

At the end of each work package, provide:

```markdown
## Outcome
<What now works for the user/business>

## PRD requirements addressed
- <IDs>

## Changes
- <Key components/contracts/migrations/UI>

## Verification
- `<command>` — PASS/FAIL

## Decisions
- <ADR or owner decision>

## Known limitations or risks
- <Only material items>

## Repository state
- Branch:
- Commit (if authorized):
- Working tree:

## Recommended next work package
<One bounded next step>
```

Do not claim “complete” when tests were not run, acceptance criteria remain unmet, or required decisions are unresolved.

---

## 25. Suggested First Claude Code CLI Prompt

Start Claude Code from the repository root after placing `PRD.md` and `CLAUDE.md` there. Use Plan Mode, then enter:

> Read `CLAUDE.md` and `PRD.md` completely. Inspect the repository and git status without generating application code. In Plan Mode, create a proposed Milestone 0 work-package plan that maps tasks to PRD requirement IDs, recommends the smallest initial deployable boundaries, lists the ADRs and owner decisions required before scaffolding, defines the durable execution-state files, and provides verification commands. Use business-oriented names. Do not create all target services, and do not add Kafka, Redis, Kubernetes, a separate vector database, or cloud resources. Wait for my approval after presenting the plan.

After the plan is approved, use:

> Exit Plan Mode and execute only the approved first work package. Maintain `Docs/Execution/CURRENT-WORK.md` throughout the session. Make small verified changes, preserve all user-owned work, and stop for unresolved material decisions. Before compaction or session exit, write a resumable checkpoint. At completion, provide the handoff format required by `CLAUDE.md` and recommend only one next bounded work package.

Recommended CLI sequence:

```bash
cd diagnostic-intelligence-platform
claude
```

Inside Claude Code, select Plan Mode before sending the first prompt. Do not start the first session with unrestricted permission bypasses.

---

## 26. Agent Self-Check

Before every material action, ask:

- Is this inside the approved work package?
- Which PRD requirement does it satisfy?
- Is this a product fact or an unapproved assumption?
- Does it preserve business capability naming?
- Does it violate a service or schema ownership boundary?
- Am I adding technology without a proven requirement?
- Is the data entirely synthetic and appropriately licensed?
- Can the deterministic system work without the LLM?
- Have authorization and failure behavior been considered?
- How will this be verified?
- Can another agent resume from durable state?

If the answer is unclear, investigate or ask before expanding the change.
