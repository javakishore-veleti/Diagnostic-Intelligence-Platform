# Diagnostic Intelligence Platform

**A healthcare digital diagnostics platform that helps members access laboratory services while enabling operations teams to manage orders, specimens, results, exceptions, and service insights.**

> **Demonstration project.** Every person, order, specimen, and result here is synthetic. Built for education and portfolio demonstration only — not a clinical product, no diagnostic or treatment recommendations, and no claim of HIPAA compliance, CLIA certification, or FDA clearance. Vendor-neutral, and not derived from any real laboratory company's systems, data, or documents.

---

## The business problem

A diagnostic laboratory runs on a simple promise: a sample is collected, it moves through the lab, and a trustworthy result comes back on time. The value is created — and lost — in the gap between those events.

Two things go wrong in that gap.

**Members are left in the dark.** An order is placed and then goes quiet. Where is my sample? Is something wrong? Is the result ready? Absent an answer, people call — and each call costs the business more than the test.

**Operations teams work blind across systems.** When a specimen stalls, the answer lives in fragments: order state here, specimen custody there, an exception code somewhere else, and the actual procedure buried in a document nobody can find quickly. Investigating one delayed specimen means assembling that story by hand, over and over, hundreds of times a day.

This platform closes both gaps. It gives members a clear, honest view of their own diagnostic journey, and it gives operations a single place to see the whole picture — with an assistant that assembles the evidence and cites the procedure, so a five-minute investigation becomes a five-second one.

---

## What the platform delivers

### For members — the diagnostic journey, visible

Browse available tests, see what was ordered and why, follow a specimen from collection through transit and processing, and read released results with reference ranges and flags in plain language. No phone call required.

### For laboratory operations — investigation, not archaeology

See every order and specimen end to end, triage exceptions and delays as they emerge, search across the whole operation, administer the test catalog and the knowledge library, generate demonstration datasets, and review the quality of the intelligence layer itself.

### For both — an assistant that shows its work

Ask *why is this specimen delayed, and what does the procedure say to do next?* and get an answer that combines live operational facts with the relevant, cited section of the standard operating procedure.

The assistant is deliberately constrained, and that constraint is the product. Authoritative facts — order state, specimen timeline, result values, who is allowed to see what, when things happened — come from the deterministic services that own them, never from the model. The assistant reads; it never writes. It explains and cites; it is never the system of record. If evidence is missing or conflicting, it says so rather than inventing a plausible answer. And it does not diagnose, recommend treatment, or replace a clinician.

The result is an intelligence layer you can put in front of an operations team without asking them to trust a black box: every answer traces back to the exact data and document version that produced it.

---

## The data

Everything the platform runs on is synthetic, reproducible, and openly sourced — the trust story is part of the product.

| What | Where it comes from |
|---|---|
| Synthetic people and their clinical histories | **Synthea** FHIR R4, generator version pinned, provenance recorded |
| Laboratory terminology | **LOINC**, used under its license, release version recorded, only required subsets imported |
| Test catalog | Project-authored, using project codes such as `LAB-10001`, mapped to LOINC |
| Orders, specimens, results, events, exception scenarios | Project-owned generator — deterministic and repeatable at configurable volumes |
| Procedures and workflow guidance | Project-authored synthetic SOPs, clearly labeled as synthetic |
| Domain context | Approved public references, stored with source and terms metadata |

Real patient data, protected health information, prior-employer material, proprietary laboratory documents, and bulk-scraped commercial content are prohibited — as a rule enforced in review, not just a preference.

The data model follows the shape of the real domain: a **customer** places a **laboratory order** for one or more **tests**; a **specimen** is collected against that order and moves through custody and processing; **results** are produced, validated, and released; and every meaningful transition emits an immutable **operational event**. Exceptions and delays are *derived* from that authoritative state and versioned rules — computed facts, not model opinions.

---

## The applications

Two portals, each served by its own experience layer, over a set of business capabilities that own their data.

**DiagnosticServices** — the member portal. Catalog, orders, specimen progress, released results, and the assistant.

**DiagnosticsAdmin** — the operations portal. Operational dashboard, cross-capability search, investigation workspace, catalog and knowledge administration, synthetic data jobs, and intelligence-quality reporting.

Behind them:

- **Laboratory services** own the record — customers, test catalog, orders, specimens, and results. Each owns its data outright.
- **Diagnostic intelligence** provides the assistant, the knowledge library and retrieval, deterministic operational insights, and the evaluation harness that measures answer quality.
- **Experience services** compose and authorize for one portal each. A portal never reaches past its experience API, and an experience service never owns laboratory data.

Three rules keep this honest as it grows: portals stay behind their experience API; every capability owns its schema and nobody reads anyone else's tables; and folders are named for what they do for the business (`KnowledgeServices`) rather than how they are built (`RagService`).

---

## The technology

Production-style choices, each made for a reason.

**Applications and services** — Angular and TypeScript for both portals in a single workspace. Java 21 LTS with Spring Boot 3.x for the business services. Python for the AI and knowledge services, where the ecosystem is strongest.

**Data** — PostgreSQL as the primary store, with capability-owned logical schemas and Flyway migrations. **pgvector** provides vector search *inside* the same database, so retrieval works without operating a separate vector store on day one.

**Serving the model** — **vLLM** hosts the open-weight generative model, exposing an **OpenAI-compatible endpoint**. Continuous batching and paged attention are what make concurrent streaming responses viable on a single GPU, and the project measures that directly: a plain Hugging Face inference baseline is benchmarked separately, then compared against vLLM on time-to-first-token, inter-token latency, tokens per second, and throughput under concurrency.

**Keeping the model replaceable** — every service reaches the model through an internal OpenAI-compatible abstraction. vLLM is the initial runtime; OpenAI, Azure OpenAI, or Amazon Bedrock can take its place through an adapter without touching a single portal API or any laboratory domain logic. Model, prompt, retrieval, and safety-policy versions are all recorded with each execution.

**Grounding the answers** — documents are versioned, parsed, chunked, and embedded; retrieval runs against published knowledge versions only and preserves source, document version, section, chunk ID, and score, so every citation is real and traceable. Responses stream over Server-Sent Events with ordered, monotonic sequence numbers.

**Contracts and operations** — OpenAPI 3.1 contracts under version control, RFC 9457 problem details for errors, correlation IDs on every request path. OpenTelemetry, Prometheus, and Grafana for traces, metrics, and dashboards — including AI-specific telemetry across retrieval, tools, and model calls. Keycloak for identity locally. Docker Compose locally, Terraform and GitHub Actions for AWS.

**Deliberately absent** — Kafka, Redis, Kubernetes, and a standalone vector database. Each is a real tool with a real cost, and each is introduced only when a documented requirement and an accepted decision record justify it. Restraint is a design goal here, not an oversight.

---

## Repository structure

```text
diagnostic-intelligence-platform/
├── PRD.md                          Product requirements — the source of truth
├── CLAUDE.md                       Operating guide for coding agents
├── REQUIREMENTS-ARCHITECTURE.md    Architecture constraints and views
├── LOCAL-DEVELOPMENT.md            Setup, ports, profiles, troubleshooting
├── SECURITY.md                     Security posture and reporting
│
├── Portals/                        The two user experiences
│   ├── DiagnosticServices/           Member portal
│   ├── DiagnosticsAdmin/             Operations portal
│   └── SharedExperience/             Shared UI library
│
├── Middleware/
│   ├── ExperienceServices/         Composition and authorization, one per portal
│   ├── LaboratoryServices/         Systems of record — customers, catalog,
│   │                                 orders, specimens, results, notifications
│   ├── DiagnosticIntelligence/     Assistant, knowledge and retrieval,
│   │                                 operational insights, evaluation quality
│   └── SharedServices/             Cross-cutting contracts and conventions only
│
├── DataMgmt/
│   ├── SyntheticDataGen/           Reproducible people, orders, specimens, results
│   ├── ReferenceData/              LOINC, units, value sets
│   ├── DataIngestion/              Synthea and reference-data pipelines
│   ├── KnowledgeData/              Synthetic SOPs, public references, eval corpus
│   └── Database/                   Schemas, migrations, seed data
│
├── DevOps/
│   ├── Local/                      One-command local stack and compose profiles
│   ├── Containers/  Terraform/  AWS/
│
├── Testing/                        Functional, Integration, Contract, EndToEnd,
│                                   Performance, Security, Resilience,
│                                   IntelligenceQuality
└── Docs/
    ├── Architecture/  ADR/  API/  DataModel/  ThreatModels/  Runbooks/
    └── Execution/                  Durable work state for long-running agents
```

Capability folders appear when their milestone begins, carrying a short boundary README until then. The tree is never scaffolded as empty modules ahead of the work.

---

## Roadmap

| Phase | What it delivers |
|---|---|
| **1 — Deterministic foundation** | Both portals and their experience APIs, test catalog, orders and specimens, synthetic data, search and detail views, deterministic timelines and delay calculation, one-command local stack, contracts, migrations, observability baseline, CI |
| **2 — Results and grounded knowledge** | Customer and result capabilities, Synthea ingestion, LOINC mapping, synthetic SOPs, chunking and embeddings, pgvector retrieval with citations, a knowledge-grounded assistant, evaluation baseline |
| **3 — Tool-assisted intelligence** | Read-only tool calling across capabilities, evidence-backed delay investigation, streamed responses, conversation history, full traces across retrieval, tools, and model |
| **4 — Cloud and LLMOps** | Terraform-managed AWS, build/scan/publish/deploy/rollback pipelines, managed PostgreSQL, GPU-backed model serving, evaluation gates and operational dashboards |

Phase 1 is designed to be genuinely useful *without* generative AI. The intelligence layer augments a platform that already works on its own.

---

## Status

**Pre-implementation.** The repository currently holds the product requirements and the agent operating guide. Application code, the local stack, and the supporting documents referenced above are not yet created — Milestone 0 settles the foundational decisions that must precede any scaffolding.

---

## Running it locally

Once the stack exists, the whole environment comes up through five scripts that work from any directory:

```bash
./DevOps/Local/docker-all-up.sh      core      # database + identity
./DevOps/Local/docker-all-status.sh            # container state and readiness
./DevOps/Local/docker-all-logs.sh              # follow logs, filter by component
./DevOps/Local/docker-all-down.sh              # stop, data preserved
```

Profiles: `core` (PostgreSQL and Keycloak), `intelligence` (model runtime and embeddings), `observability` (collector, metrics, dashboards), `all`. The GPU-dependent model runtime stays optional — a lightweight OpenAI-compatible stub or a configured remote provider keeps development moving without a GPU. Services can run from an IDE while infrastructure runs in containers.

---

## Documentation

| Document | Purpose |
|---|---|
| `PRD.md` | Product requirements, personas, capabilities, requirement IDs, acceptance criteria |
| `CLAUDE.md` | Operating guide for Claude Code and other coding agents |
| `REQUIREMENTS-ARCHITECTURE.md` | Architecture constraints and views |
| `Docs/ADR/` | Accepted architecture decisions |
| `Docs/API/` | Versioned OpenAPI and event contracts |

Requirement IDs (`FR-AST-004`, `NFR-PERF-001`, …) are stable and referenced from issues, tests, and pull requests.

---

## License

MIT — see [LICENSE](LICENSE).
