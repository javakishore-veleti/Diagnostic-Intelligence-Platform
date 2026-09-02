# Diagnostic Intelligence Platform

## Product Requirements Document (PRD)

**Document status:** Implementation-ready baseline  
**Version:** 1.0  
**Date:** September 2, 2026  
**Intended users of this document:** Product owner, solution architect, developers, Claude Code, Cursor, and other coding agents  
**Repository model:** Single GitHub monorepo  
**Initial deployment target:** Local Docker-based development, followed by AWS  

---

## 1. Executive Summary

The Diagnostic Intelligence Platform is a portfolio-grade, healthcare laboratory operations and diagnostic-services application built with public standards, public reference material used within applicable terms, and entirely synthetic patient and operational data.

The platform will provide two web experiences:

1. **DiagnosticServices** — a customer-facing portal where a synthetic customer can view laboratory orders, specimen progress, results, and grounded plain-language explanations.
2. **DiagnosticsAdmin** — an internal operations portal where laboratory staff can manage tests, monitor orders and specimens, investigate delays and exceptions, manage knowledge content, generate synthetic data, evaluate intelligent-assistant quality, and observe platform health.

The system will combine conventional deterministic application behavior with a bounded intelligence capability. Transactional facts such as order status, specimen status, test results, user authorization, and timestamps must always come from authoritative application services. Large language models may summarize, explain, retrieve supporting knowledge, and orchestrate approved read-only tools, but they must not invent, alter, or independently diagnose clinical conditions.

The project is intended to demonstrate modern hands-on engineering across Java, Spring Boot, Angular, Python, PostgreSQL, pgvector, Redis, event-driven integration, OpenTelemetry, vLLM, RAG, tool calling, AI evaluation, Docker, GitHub Actions, Terraform, and AWS. The implementation must grow incrementally; the target repository may show the final capability boundaries, but Phase 1 must not begin as an unnecessarily large collection of deployable microservices.

---

## 2. Product Vision

Build a modern diagnostic laboratory services platform that makes laboratory operations understandable, traceable, observable, and easier to investigate while demonstrating how generative AI can safely operate over structured laboratory data and curated knowledge.

The product should answer questions such as:

- What laboratory orders belong to this customer?
- Where is a specimen in its processing lifecycle?
- Why is an order delayed?
- Which factual events and operating guidance support that explanation?
- What does a result mean in plain language, without providing a medical diagnosis?
- Which tests are available, what specimens do they require, and what is the expected turnaround time?
- How well is the intelligent assistant retrieving, grounding, and explaining information?

---

## 3. Problem Statement

Laboratory information is commonly distributed across patient records, order systems, specimen workflows, result systems, test catalogs, operational events, and procedural documents. A customer wants a simple, trustworthy view of their orders and results. An operations user needs a deeper view that connects an order to its specimens, events, expected turnaround time, exceptions, and relevant operating guidance.

Traditional portals present these areas independently and force users to reconstruct the full state manually. A generic chatbot is also insufficient because it may lack access to authoritative data, produce unsupported answers, or blur the boundary between operational explanation and medical diagnosis.

This product addresses the problem with:

- Business-capability-aligned services and portals.
- Traceable synthetic laboratory workflows.
- Searchable structured and unstructured information.
- Grounded answers with citations to project-owned knowledge.
- Approved tool calls to authoritative services.
- Streaming responses with deterministic event ordering.
- AI quality evaluation and operational observability.

---

## 4. Product Naming and Vocabulary

### 4.1 Product name

The working product and repository name is:

`diagnostic-intelligence-platform`

The name must not include Quest, a past employer, or another real laboratory company.

### 4.2 Naming rule

Top-level repository folders and deployable business capabilities use business language. Technical terms belong inside the implementation of those capabilities.

Examples:

- Use `DiagnosticIntelligence/KnowledgeServices`, not `RagService` as a top-level capability.
- Use `DiagnosticIntelligence/AssistantServices`, not `VllmService` as the business capability.
- Use `DevOps/Local/ModelRuntime/vLLM` for the technical runtime.
- Use `Testing/IntelligenceQuality` for AI evaluation assets.

### 4.3 Canonical business terms

| Term | Meaning |
|---|---|
| DiagnosticServices | Customer-facing digital experience and its experience API |
| DiagnosticsAdmin | Internal operations and administration experience and its experience API |
| LaboratoryServices | Business services for customers, orders, specimens, tests, and results |
| DiagnosticIntelligence | Assistant, knowledge, insight, and intelligence-quality capabilities |
| DataMgmt | Synthetic generation, ingestion, reference data, schemas, and knowledge data |
| Customer | A synthetic person using DiagnosticServices; represented using appropriate FHIR resources where applicable |
| Laboratory order | A request for one or more laboratory tests |
| Specimen | Collected material associated with an order and one or more tests |
| Result | A synthetic laboratory observation or panel result |
| Operational event | An immutable business event describing progress or an exception |
| Knowledge source | An approved document used for retrieval and grounded answers |

---

## 5. Goals

### 5.1 Product goals

- Provide a customer portal for synthetic orders, specimen progress, and results.
- Provide an operations portal for end-to-end order and specimen investigation.
- Provide a grounded assistant that combines structured service data with curated knowledge.
- Demonstrate deterministic, auditable tool orchestration rather than an unconstrained chatbot.
- Provide complete traceability from an answer to tool results and knowledge sources.
- Support realistic synthetic laboratory data at configurable volumes.
- Support local development through simple Docker orchestration scripts.
- Provide a path from local development to automated AWS deployment.
- Measure conventional software quality and AI-specific quality.
- Keep model providers replaceable through an OpenAI-compatible model-access abstraction.

### 5.2 Learning and portfolio goals

- Build production-style Spring Boot APIs and business services.
- Build two Angular portals in a monorepo.
- Apply FHIR concepts, LOINC terminology, and laboratory workflow modeling.
- Compare baseline Hugging Face inference with vLLM separately, then integrate vLLM through an OpenAI-compatible API.
- Implement streaming, RAG, embeddings, tool calling, evaluation, and LLM observability.
- Implement CI/CD, infrastructure as code, containerization, security controls, and operational dashboards.
- Preserve hands-on code ownership rather than producing only an architecture demonstration.

---

## 6. Non-Goals and Safety Boundaries

The platform must not:

- Diagnose disease, recommend treatment, prescribe medication, or replace a clinician.
- Use real patient data, protected health information, production claims, or employer data.
- Copy or recreate proprietary Quest Diagnostics systems, code, data, internal workflows, or documents.
- Present itself as a Quest product or use Quest names, marks, logos, proprietary test identifiers, or visual branding.
- Scrape an entire commercial test directory or ingest content contrary to its terms.
- Claim HIPAA compliance, CLIA certification, FDA clearance, or suitability for real clinical use.
- Permit an LLM to directly mutate orders, specimens, results, test definitions, user permissions, or audit records.
- Treat an LLM response as the system of record.
- Introduce Kafka, Kubernetes, a separate vector database, or many deployables before a verified requirement justifies them.
- Implement billing, insurance adjudication, claims processing, appointment scheduling, provider ordering, or real notifications in the initial release.

Every portal must display an appropriate notice that all people, orders, specimens, and results are synthetic and the product is for demonstration and educational use only.

---

## 7. Users and Personas

### 7.1 Synthetic customer

The customer wants to:

- Sign in to a demonstration account.
- View their profile and recent laboratory orders.
- Track specimen and order progress.
- View released synthetic results.
- Read understandable, grounded explanations.
- Ask questions about their own records and approved educational content.

The customer must never see another customer's data or internal operational notes.

### 7.2 Laboratory operations specialist

The operations specialist wants to:

- Search by order ID, specimen ID, synthetic customer ID, or correlation ID.
- Inspect the complete order/specimen timeline.
- Compare expected and actual turnaround time.
- Identify stalled or failed workflow stages.
- View related catalog rules and operating procedures.
- Ask the assistant why an item is delayed and receive an evidence-backed explanation.

### 7.3 Laboratory catalog administrator

The catalog administrator wants to:

- Create and maintain project-owned test definitions.
- Associate tests with LOINC codes, specimen requirements, units, reference ranges, and expected turnaround time.
- Retire test definitions without destroying history.
- Validate reference-data import results.

### 7.4 Knowledge curator

The knowledge curator wants to:

- Upload or register approved documents.
- Review parsing and chunking results.
- publish, supersede, or retire knowledge versions.
- Re-index content.
- Test retrieval before making content available to the assistant.

### 7.5 Intelligence quality analyst

The quality analyst wants to:

- Manage evaluation datasets.
- Run retrieval and end-to-end answer evaluations.
- Compare model, prompt, retrieval, and configuration versions.
- Inspect unsupported claims, failed citations, latency, and token usage.
- prevent a lower-quality configuration from being promoted.

### 7.6 Platform administrator/developer

The platform user wants to:

- Start, stop, inspect, and troubleshoot the local stack.
- View health, metrics, traces, and logs.
- Generate or reset synthetic demonstration datasets.
- Run tests and deploy through controlled GitHub Actions workflows.

---

## 8. Product Scope by Release

### 8.1 MVP / Phase 1 — deterministic laboratory foundation

Phase 1 delivers a useful system without depending on generative AI.

- Monorepo foundation and build conventions.
- DiagnosticServices and DiagnosticsAdmin Angular shells.
- DiagnosticServices experience API and DiagnosticsAdmin experience API.
- Test catalog, laboratory order, and specimen capabilities.
- PostgreSQL with logically separated schemas.
- Synthetic test catalog, customers, orders, specimens, and events.
- Search and detail views.
- Deterministic order/specimen timeline and delay calculation.
- Local Docker scripts and health/status output.
- OpenAPI contracts, migrations, tests, observability baseline, and CI checks.

### 8.2 Phase 2 — results and grounded knowledge

- Customer and result capabilities.
- Synthea FHIR R4 ingestion.
- LOINC reference-data ingestion and mapping.
- Curated project-owned synthetic SOPs and approved public references.
- Document lifecycle, chunking, embeddings, pgvector retrieval, and citations.
- Assistant capable of knowledge-only grounded question answering.
- AI evaluation baseline.

### 8.3 Phase 3 — tool-assisted diagnostic intelligence

- Approved read-only tool calling across order, specimen, result, catalog, and knowledge capabilities.
- Evidence-backed delay investigation.
- Streaming assistant responses with ordered events.
- Conversation persistence with retention controls.
- Redis for justified cache/session use.
- Event-driven workflows and Kafka only for documented use cases.
- Complete OpenTelemetry traces across assistant, tools, retrieval, and model calls.

### 8.4 Phase 4 — AWS and LLMOps

- Terraform-managed AWS environments.
- GitHub Actions build, scan, publish, deploy, verify, and rollback workflows.
- Managed PostgreSQL and appropriate container hosting.
- GPU-backed vLLM deployment when cost and environment permit; provider fallback configuration otherwise.
- Performance, resilience, security, and cost tests.
- Model/prompt/retrieval evaluation gates and operational dashboards.

---

## 9. Business Capability Map

### 9.1 ExperienceServices

- `DiagnosticsAdminServices` serves only the DiagnosticsAdmin portal.
- `DiagnosticServicesApi` serves only the DiagnosticServices portal.
- Experience services compose data from business services, enforce experience-specific authorization, shape responses, and expose streaming endpoints.
- Portals must not call laboratory microservices directly.
- Experience services must not own laboratory systems-of-record data.

### 9.2 LaboratoryServices

- `CustomerServices` owns synthetic customer identity/profile projections used by this product.
- `LabOrderServices` owns laboratory orders, ordered tests, order state, and order-level events.
- `SpecimenServices` owns specimens, specimen requirements, accession-like project identifiers, specimen state, and specimen events.
- `TestCatalogServices` owns project test definitions and mappings to public terminology.
- `ResultServices` owns synthetic results, panels, values, units, ranges, flags, status, and release state.

The final repository contains these capability locations. The implementation may initially combine closely related capabilities into fewer deployable Spring Boot applications, provided module boundaries and data ownership are preserved and documented in an ADR.

### 9.3 DiagnosticIntelligence

- `AssistantServices` owns conversation orchestration, approved tool selection, response streaming, safety policies, prompts, and model access.
- `KnowledgeServices` owns knowledge documents, versions, parsing, chunking, embeddings, retrieval, and citation metadata.
- `InsightsServices` owns deterministic operational insights such as delayed specimens, turnaround-time trends, exception aggregation, and later assisted summaries.
- `QualityServices` owns evaluation definitions, evaluation runs, scores, comparisons, and release recommendations.

### 9.4 SharedServices

Shared code must be small and stable. It may include cross-cutting security helpers, observability conventions, error envelopes, correlation identifiers, test fixtures, and generated API clients. It must not become a dumping ground for domain models or shared database entities.

---

## 10. Functional Requirements

Requirement IDs are stable and must be referenced in issues, tests, and pull requests.

### 10.1 Authentication and authorization

- **FR-IAM-001:** The local platform shall authenticate users through a locally runnable identity provider, initially Keycloak.
- **FR-IAM-002:** The platform shall support at least `CUSTOMER`, `OPERATIONS`, `CATALOG_ADMIN`, `KNOWLEDGE_CURATOR`, `QUALITY_ANALYST`, and `PLATFORM_ADMIN` roles.
- **FR-IAM-003:** A customer shall access only records explicitly associated with that synthetic customer identity.
- **FR-IAM-004:** Administrative functions shall enforce role-based access in APIs, not only in the UI.
- **FR-IAM-005:** Services shall propagate authenticated subject, roles, correlation ID, and trace context.
- **FR-IAM-006:** Authorization failures shall return a consistent error contract without leaking record existence.

### 10.2 DiagnosticServices portal

- **FR-CXP-001:** The portal shall provide sign-in, sign-out, session-expiration, and access-denied experiences.
- **FR-CXP-002:** The home dashboard shall show the customer's recent orders and a concise status summary.
- **FR-CXP-003:** The customer shall filter and paginate their laboratory orders.
- **FR-CXP-004:** An order-detail page shall show ordered tests, linked specimens, ordered timestamps, current status, and a chronological timeline.
- **FR-CXP-005:** A result view shall show only released results.
- **FR-CXP-006:** Result presentation shall show test name, value, unit, synthetic reference range, flag, collection time, result time, and release status when applicable.
- **FR-CXP-007:** The portal shall visibly label all records as synthetic demonstration data.
- **FR-CXP-008:** The assistant shall limit customer questions and tool access to the authenticated customer's records and approved general knowledge.
- **FR-CXP-009:** Assistant citations shall be selectable and reveal the supporting source title, section, and version.
- **FR-CXP-010:** Explanations shall include a non-diagnostic disclaimer and direct users to a qualified healthcare professional for medical interpretation.

### 10.3 DiagnosticsAdmin portal

- **FR-ADM-001:** The operations dashboard shall show counts of orders and specimens by state, delayed items, exception categories, and recent failures.
- **FR-ADM-002:** Authorized users shall search by project customer ID, order ID, specimen ID, test code, and correlation ID.
- **FR-ADM-003:** Search results shall support filters, sorting, pagination, and deep links.
- **FR-ADM-004:** The investigation view shall join authorized read models for customer, order, specimen, result, test catalog, and operational events without bypassing service ownership.
- **FR-ADM-005:** The view shall distinguish observed facts, deterministic rules, retrieved guidance, and model-generated explanation.
- **FR-ADM-006:** The platform shall calculate expected versus actual elapsed time and identify the first overdue stage using deterministic rules.
- **FR-ADM-007:** Users shall see the complete append-only event timeline and applicable correlation/trace identifiers.
- **FR-ADM-008:** The assistant shall answer operational questions using approved read-only tools and cite both structured evidence and knowledge sources.
- **FR-ADM-009:** The portal shall allow authorized catalog management.
- **FR-ADM-010:** The portal shall allow authorized knowledge lifecycle management.
- **FR-ADM-011:** The portal shall allow authorized synthetic dataset generation and show generation job status.
- **FR-ADM-012:** The portal shall provide intelligence-quality run summaries and configuration comparisons.
- **FR-ADM-013:** The portal shall provide links or embedded views for system health and observability dashboards where configured.

### 10.4 Customer capability

- **FR-CUS-001:** The platform shall import synthetic FHIR R4 Patient resources from an approved Synthea dataset.
- **FR-CUS-002:** Import shall be idempotent and retain source and import-batch provenance.
- **FR-CUS-003:** Customer records shall use project-generated identifiers externally and shall not expose unnecessary source identifiers.
- **FR-CUS-004:** Customer search fields shall be deliberately limited in logs and administrative responses.
- **FR-CUS-005:** A customer projection may be deactivated without deleting linked historical records.

### 10.5 Test catalog capability

- **FR-CAT-001:** The catalog shall support project-owned test codes, display names, descriptions, active dates, status, expected turnaround time, specimen requirements, methods, units, reference-range definitions, and LOINC mappings.
- **FR-CAT-002:** A test definition shall have immutable version history once used by an order.
- **FR-CAT-003:** Catalog changes shall be audited.
- **FR-CAT-004:** LOINC imports shall store release/version provenance and licensing metadata.
- **FR-CAT-005:** Mapping validation shall report unknown, ambiguous, duplicated, and retired codes.
- **FR-CAT-006:** Retiring a test shall prevent new orders while retaining historical readability.

### 10.6 Laboratory order capability

- **FR-ORD-001:** The platform shall create synthetic laboratory orders containing one or more ordered tests.
- **FR-ORD-002:** Each order shall have a unique opaque project identifier and an idempotency key for creation.
- **FR-ORD-003:** Supported baseline states shall include `DRAFT`, `SUBMITTED`, `COLLECTION_PENDING`, `IN_PROGRESS`, `PARTIALLY_COMPLETED`, `COMPLETED`, `CANCELLED`, and `ON_HOLD`.
- **FR-ORD-004:** State transitions shall be validated by deterministic rules.
- **FR-ORD-005:** Every accepted state transition shall append an operational event.
- **FR-ORD-006:** Orders shall reference versioned catalog definitions so history remains reproducible.
- **FR-ORD-007:** Order reads shall support customer, date, status, test, and exception filters.

### 10.7 Specimen capability

- **FR-SPC-001:** An order may require one or more specimens; a specimen may support one or more ordered tests when compatible.
- **FR-SPC-002:** Baseline specimen states shall include `EXPECTED`, `COLLECTED`, `IN_TRANSIT`, `RECEIVED`, `ACCESSIONED`, `PROCESSING`, `ANALYZED`, `REVIEW_PENDING`, `COMPLETED`, `REJECTED`, and `CANCELLED`.
- **FR-SPC-003:** Specimen events shall record event type, event time, recorded time, source, location, actor/system, reason, and correlation ID.
- **FR-SPC-004:** Events shall be append-only; corrections shall be represented by compensating events.
- **FR-SPC-005:** Deterministic exception rules shall identify missing collection, transport delay, processing delay, rejection, result delay, and invalid transition conditions.
- **FR-SPC-006:** Expected completion shall derive from the applicable versioned test/catalog and workflow rule.
- **FR-SPC-007:** A specimen timeline shall remain stable and consistently ordered by event time, recorded time, and event ID.

### 10.8 Result capability

- **FR-RES-001:** Results shall be synthetic and associated with an ordered test and specimen where applicable.
- **FR-RES-002:** Results shall support scalar numeric, coded, textual, and panel structures.
- **FR-RES-003:** Numeric results may include unit, lower/upper synthetic range, and deterministic `LOW`, `NORMAL`, `HIGH`, or `CRITICAL` flags.
- **FR-RES-004:** Result states shall include `PRELIMINARY`, `FINAL`, `CORRECTED`, `CANCELLED`, and `WITHHELD`.
- **FR-RES-005:** Only authorized roles and the associated customer may view released `FINAL` or `CORRECTED` results.
- **FR-RES-006:** A correction shall preserve the superseded value and audit trail.
- **FR-RES-007:** Plain-language explanations shall preserve the exact authoritative value and unit and must not state a diagnosis.

### 10.9 Synthetic data management

- **FR-DAT-001:** A command-line and administrative API capability shall generate repeatable datasets using an explicit random seed.
- **FR-DAT-002:** Generation profiles shall include `tiny`, `demo`, `performance`, and user-defined counts.
- **FR-DAT-003:** The generator shall create internally consistent customers, tests, orders, specimens, events, and results.
- **FR-DAT-004:** The generator shall inject configurable exception scenarios including delays, missing events, rejection, re-collection, corrected results, and partial panels.
- **FR-DAT-005:** Every generated record shall carry dataset ID, generator version, seed, and generation timestamp.
- **FR-DAT-006:** Dataset generation shall validate referential and workflow consistency before marking a dataset ready.
- **FR-DAT-007:** Reset/destructive operations shall require an explicit local/demo environment guard and confirmation flag.
- **FR-DAT-008:** Synthea ingestion, LOINC ingestion, synthetic operations generation, and knowledge ingestion shall be separate jobs.
- **FR-DAT-009:** Failed jobs shall be restartable or safely repeatable without duplicate records.
- **FR-DAT-010:** The platform shall publish a machine-readable dataset manifest with counts, versions, checksums, and validation results.

### 10.10 Knowledge management and retrieval

- **FR-KNW-001:** Knowledge sources shall support project-owned synthetic SOPs, public-domain documents, properly licensed references, and authored educational content.
- **FR-KNW-002:** Every source shall store title, source type, origin, license/terms note, owner, effective date, version, approval state, and checksum.
- **FR-KNW-003:** Lifecycle states shall include `DRAFT`, `VALIDATED`, `PUBLISHED`, `SUPERSEDED`, `RETIRED`, and `FAILED`.
- **FR-KNW-004:** Only published versions shall be retrievable in end-user assistant flows.
- **FR-KNW-005:** Ingestion shall preserve page, section, heading, and source offsets sufficient to create useful citations.
- **FR-KNW-006:** Chunking and embedding configuration shall be versioned.
- **FR-KNW-007:** Reprocessing a source shall not silently overwrite the previous published version.
- **FR-KNW-008:** Retrieval shall support semantic search and metadata filters; hybrid lexical search may be added when evaluation demonstrates value.
- **FR-KNW-009:** Retrieval results shall include score, source/version identifiers, chunk identifiers, and citation metadata.
- **FR-KNW-010:** Curators shall preview retrieval results for a test query before publication.
- **FR-KNW-011:** Content originating from the public web shall not be bulk scraped or redistributed without an approved basis.

### 10.11 Assistant and model access

- **FR-AST-001:** The assistant shall access models through an internal OpenAI-compatible model-access abstraction.
- **FR-AST-002:** The initial self-hosted provider shall be vLLM; provider-specific details shall not leak into portal contracts.
- **FR-AST-003:** The assistant shall support knowledge-only and tool-assisted modes.
- **FR-AST-004:** Approved tools shall be explicitly registered with schemas, authorization rules, timeouts, retry policies, and maximum result sizes.
- **FR-AST-005:** Initial tools shall be read-only: get customer summary, get order, list customer orders, get specimen, get timeline, get results, get test definition, calculate delay facts, and search knowledge.
- **FR-AST-006:** Tool authorization shall be enforced by the target service and never delegated to the model.
- **FR-AST-007:** The orchestrator shall cap tool-call depth, number of calls, context size, output size, and execution time.
- **FR-AST-008:** Tool results shall be treated as untrusted model input and protected against prompt injection.
- **FR-AST-009:** The final answer shall distinguish verified facts, retrieved guidance, and uncertainty.
- **FR-AST-010:** Operational conclusions shall cite supporting structured records; knowledge claims shall cite published sources.
- **FR-AST-011:** When evidence is missing or conflicting, the assistant shall say what could not be determined rather than infer an unsupported fact.
- **FR-AST-012:** The assistant shall refuse diagnosis, treatment, real-patient use, or attempts to override its safety boundary.
- **FR-AST-013:** Prompts, tool definitions, model settings, retrieval settings, and safety policy versions shall be recorded for each assistant execution.
- **FR-AST-014:** Conversations shall have configurable retention and deletion policies and shall not be used for model training by default.
- **FR-AST-015:** A deterministic response template shall be available when the model runtime is unavailable.

### 10.12 Streaming contract

- **FR-STR-001:** Assistant endpoints shall stream using Server-Sent Events initially; WebSocket support is out of scope unless bidirectional behavior becomes necessary.
- **FR-STR-002:** Every stream event shall include `conversationId`, `requestId`, monotonically increasing `sequence`, `eventType`, `timestamp`, and typed payload.
- **FR-STR-003:** Supported event types shall include `request.accepted`, `status`, `tool.started`, `tool.completed`, `retrieval.completed`, `answer.delta`, `citation`, `answer.completed`, and `error`.
- **FR-STR-004:** Event order shall be deterministic even when independent work executes concurrently.
- **FR-STR-005:** The server shall emit tool events according to the declared plan/order and buffer concurrent completions when required to preserve the contract.
- **FR-STR-006:** `answer.delta` content shall be reconstructable by sequence order.
- **FR-STR-007:** The terminal event shall contain finish reason, complete citation list, latency breakdown, and safety status.
- **FR-STR-008:** Clients shall handle reconnect, duplicate delivery, cancellation, timeout, and partial failure.
- **FR-STR-009:** Streaming must not expose hidden prompts, chain-of-thought, credentials, or raw internal errors.

### 10.13 Intelligence quality

- **FR-IQ-001:** Evaluation records shall contain question, scenario, authorized user context, expected facts, expected sources, forbidden claims, and grading rules.
- **FR-IQ-002:** Evaluation shall measure retrieval recall/precision at K, citation correctness, groundedness, factual correctness, answer completeness, refusal correctness, tool selection, tool argument correctness, latency, throughput, and token usage.
- **FR-IQ-003:** Deterministic graders shall be used wherever possible; model-based graders must record grader model/version and be calibrated against human-reviewed examples.
- **FR-IQ-004:** Runs shall be reproducible from configuration and dataset versions.
- **FR-IQ-005:** Quality comparisons shall prevent promotion when configured critical thresholds regress.
- **FR-IQ-006:** The evaluation suite shall include prompt injection, cross-customer access, unsupported diagnosis, missing evidence, conflicting evidence, and unavailable dependency scenarios.
- **FR-IQ-007:** Evaluation results shall be visible in DiagnosticsAdmin and exportable as JSON/CSV.

### 10.14 Notifications

Real external messaging is not required initially. A later `NotificationServices` capability may generate local demonstration notifications for released results or processing exceptions. It must consume events and must not become a synchronous dependency of core state transitions.

---

## 11. Principal User Journeys

### 11.1 Customer views an order

1. Customer authenticates.
2. DiagnosticServices loads orders through `DiagnosticServicesApi`.
3. The experience API obtains only authorized customer data.
4. Customer selects an order.
5. The portal shows tests, specimen status, released results, and ordered timeline.
6. Customer may ask a bounded question about the displayed record.
7. The response streams with citations and a non-diagnostic notice.

### 11.2 Operations investigates a delay

1. Operations user searches for an order or specimen.
2. DiagnosticsAdmin displays authoritative state and timeline.
3. Deterministic insights identify overdue stages and calculate elapsed time.
4. The user asks, “Why is this specimen delayed?”
5. AssistantServices calls authorized read-only tools.
6. KnowledgeServices retrieves applicable published guidance.
7. The model produces an explanation constrained to the evidence.
8. The UI separates facts, applicable guidance, possible operational interpretation, missing information, and next operational check.
9. Each claim provides a structured-record or document citation.

### 11.3 Curator publishes a synthetic SOP

1. Curator creates a draft knowledge record and uploads an approved file.
2. KnowledgeServices performs malware/type checks, parsing, normalization, chunking, and embedding.
3. The platform reports unsupported content and processing errors.
4. Curator previews chunks and sample retrieval.
5. Curator publishes the validated version.
6. New assistant requests can retrieve it; prior versions remain auditable.

### 11.4 Quality analyst compares configurations

1. Analyst selects a versioned evaluation dataset.
2. Analyst selects baseline and candidate model/prompt/retrieval configurations.
3. QualityServices executes both under equivalent controls.
4. The portal displays metric deltas and individual failures.
5. The candidate receives a pass, warning, or blocked recommendation based on configured thresholds.

---

## 12. Data Sources and Data Policy

### 12.1 Approved initial sources

| Need | Source | Policy |
|---|---|---|
| Synthetic people and clinical records | Synthea FHIR R4 | Pin dataset/generator version; record provenance |
| Laboratory terminology | LOINC | Follow license; record release/version; import only required subsets initially |
| Laboratory operational records | Project SyntheticDataGen | Entirely project-owned and reproducible |
| Test catalog | Project-authored synthetic catalog mapped to LOINC | Use project codes such as `LAB-10001` |
| SOPs and workflow guidance | Project-authored synthetic documents | Clearly label synthetic/educational |
| Domain context | Approved public references | Store source/terms metadata; do not bulk scrape |
| Claims, later only | CMS synthetic datasets | Separate future scope and ingestion pipeline |

### 12.2 Prohibited sources

- Real patient or employee data.
- Data retained from prior employment.
- Proprietary laboratory documents or screenshots.
- Credentials, tokens, production logs, or internal API responses.
- Unlicensed bulk copies of commercial content.
- Public content whose applicable terms prohibit the intended use.

### 12.3 Initial dataset targets

The default `demo` profile should target approximately:

- 100–1,000 synthetic customers.
- 20–50 project-authored tests and panels.
- 1,000–5,000 laboratory orders.
- 1,500–8,000 specimens.
- 5,000–30,000 results.
- 10,000–100,000 operational events.
- At least 10 examples of every supported exception scenario.

These counts are configuration targets, not hard-coded values.

### 12.4 Core conceptual relationships

- A customer has zero or more laboratory orders.
- An order contains one or more ordered tests.
- An order requires one or more specimens.
- A compatible specimen can support one or more ordered tests.
- A specimen has an append-only event timeline.
- An ordered test has zero or more result versions.
- A test references a versioned catalog definition and optional LOINC mappings.
- An exception is derived from authoritative state/events and versioned rules.
- A knowledge citation references an immutable knowledge-source version and chunk.
- An assistant execution references its conversation, tool executions, retrieval set, configuration versions, and evaluation metadata.

### 12.5 Database ownership

Local environments may use one PostgreSQL instance, but logical ownership is mandatory:

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

A service must not directly query or update another capability's tables. Cross-capability access occurs through APIs, events, or deliberately owned read models. Every database change must use versioned migrations, initially Flyway for Java-owned schemas and an explicitly documented equivalent for Python-owned schemas.

---

## 13. API and Integration Requirements

### 13.1 API conventions

- REST/JSON is the default synchronous integration style.
- OpenAPI 3.1 specifications are source-controlled.
- APIs use a versioned base path such as `/api/v1`.
- IDs are opaque strings/UUIDs, not sequential database keys.
- Timestamps use ISO 8601 UTC.
- List endpoints provide pagination, filtering, stable sorting, and documented limits.
- Mutation endpoints use idempotency keys where replay is possible.
- Errors use one consistent problem-details contract based on RFC 9457.
- Every response carries or returns a correlation/request identifier.
- Generated clients may be used, but contracts remain capability-owned.

### 13.2 Initial experience API surfaces

`DiagnosticServicesApi` shall eventually expose:

- Customer profile summary.
- Customer order list and detail.
- Specimen timeline for an authorized order.
- Released result list and detail.
- Assistant conversation creation and SSE streaming.

`DiagnosticsAdminServices` shall eventually expose:

- Operations dashboard.
- Cross-capability search.
- Investigation read model.
- Catalog administration.
- Knowledge administration.
- Synthetic data job administration.
- Quality run administration and results.
- Platform health links/summaries.

Exact paths and payload schemas belong in capability OpenAPI documents; coding agents must not invent multiple overlapping endpoints without updating those contracts.

### 13.3 Events

Event-driven integration is added only when a use case requires decoupling, replay, fan-out, or asynchronous processing. Candidate events include:

- `LaboratoryOrderSubmitted`
- `SpecimenCollected`
- `SpecimenReceived`
- `SpecimenProcessingDelayed`
- `SpecimenRejected`
- `ResultFinalized`
- `ResultCorrected`
- `KnowledgeVersionPublished`
- `SyntheticDatasetReady`

Events shall use a common envelope containing event ID, type, schema version, aggregate type/ID, occurred time, recorded time, producer, correlation ID, causation ID, trace context, and payload. Producers shall use an outbox pattern when database state and event publication must be atomic. Consumers must be idempotent. Schema compatibility must be tested.

---

## 14. Repository Structure

```text
diagnostic-intelligence-platform/
├── README.md
├── PRD.md
├── REQUIREMENTS-ARCHITECTURE.md
├── LOCAL-DEVELOPMENT.md
├── SECURITY.md
├── CONTRIBUTING.md
├── LICENSE
├── Portals/
│   ├── DiagnosticsAdmin/
│   ├── DiagnosticServices/
│   └── SharedExperience/
├── Middleware/
│   ├── ExperienceServices/
│   │   ├── DiagnosticsAdminServices/
│   │   └── DiagnosticServicesApi/
│   ├── LaboratoryServices/
│   │   ├── CustomerServices/
│   │   ├── LabOrderServices/
│   │   ├── SpecimenServices/
│   │   ├── TestCatalogServices/
│   │   ├── ResultServices/
│   │   └── NotificationServices/
│   ├── DiagnosticIntelligence/
│   │   ├── AssistantServices/
│   │   │   ├── api/
│   │   │   ├── orchestration/
│   │   │   ├── tools/
│   │   │   ├── prompts/
│   │   │   ├── model-access/
│   │   │   │   ├── openai-compatible/
│   │   │   │   └── vllm/
│   │   │   └── streaming/
│   │   ├── KnowledgeServices/
│   │   │   ├── api/
│   │   │   ├── ingestion/
│   │   │   ├── parsing/
│   │   │   ├── chunking/
│   │   │   ├── embeddings/
│   │   │   ├── retrieval/
│   │   │   └── vector-store/
│   │   ├── InsightsServices/
│   │   └── QualityServices/
│   └── SharedServices/
│       ├── ApiContracts/
│       ├── Security/
│       ├── Observability/
│       └── TestSupport/
├── DataMgmt/
│   ├── SyntheticDataGen/
│   │   ├── Customers/
│   │   ├── TestCatalog/
│   │   ├── LabOrders/
│   │   ├── Specimens/
│   │   ├── Results/
│   │   ├── OperationalEvents/
│   │   ├── ExceptionScenarios/
│   │   └── Profiles/
│   ├── ReferenceData/
│   │   ├── LOINC/
│   │   ├── Units/
│   │   └── ValueSets/
│   ├── DataIngestion/
│   │   ├── Synthea/
│   │   └── ReferenceData/
│   ├── KnowledgeData/
│   │   ├── SyntheticSOPs/
│   │   ├── ApprovedPublicReferences/
│   │   ├── EvaluationCorpus/
│   │   └── Manifests/
│   └── Database/
│       ├── Schemas/
│       ├── Migrations/
│       └── SeedData/
├── DevOps/
│   ├── Local/
│   │   ├── docker-all-up.sh
│   │   ├── docker-all-down.sh
│   │   ├── docker-all-status.sh
│   │   ├── docker-all-logs.sh
│   │   ├── docker-all-reset.sh
│   │   ├── Postgres/docker-compose.yaml
│   │   ├── Redis/docker-compose.yaml
│   │   ├── Messaging/docker-compose.yaml
│   │   ├── Identity/docker-compose.yaml
│   │   ├── Observability/docker-compose.yaml
│   │   └── ModelRuntime/
│   │       └── vLLM/docker-compose.yaml
│   ├── Containers/
│   ├── Terraform/
│   │   ├── Modules/
│   │   └── Environments/
│   ├── AWS/
│   ├── Kubernetes/
│   └── Scripts/
├── Testing/
│   ├── Functional/
│   ├── Integration/
│   ├── Contract/
│   ├── EndToEnd/
│   ├── Performance/
│   ├── Security/
│   ├── Resilience/
│   └── IntelligenceQuality/
│       ├── Datasets/
│       ├── Retrieval/
│       ├── GroundedAnswers/
│       ├── ToolUse/
│       ├── Safety/
│       └── Reports/
├── Docs/
│   ├── Architecture/
│   ├── ADR/
│   ├── API/
│   ├── DataModel/
│   ├── ThreatModels/
│   └── Runbooks/
└── .github/
    ├── workflows/
    ├── actions/
    ├── ISSUE_TEMPLATE/
    └── CODEOWNERS
```

Empty placeholder services must not be generated merely to reproduce this tree. A capability folder should contain a README explaining its intended boundary until that capability is implemented.

---

## 15. Technology Baseline

Versions must be pinned at project initialization using currently supported stable/LTS releases and recorded in an ADR; this PRD does not guess future patch versions.

| Area | Baseline decision |
|---|---|
| Java services | Java 21 or newer supported LTS; Spring Boot 3.x supported release |
| Build | Maven or Gradle selected once in an ADR; one convention across Java services |
| Web portals | Current supported Angular, TypeScript, Angular workspace/library conventions |
| AI/knowledge services | Supported Python 3.x with reproducible dependency locking |
| Primary database | PostgreSQL |
| Vector search | pgvector inside PostgreSQL initially |
| Cache | Redis only when Phase 3 use cases justify it |
| Messaging | Kafka only when event use cases justify it |
| Identity | Keycloak locally; cloud identity remains replaceable |
| Model serving | vLLM exposing an OpenAI-compatible endpoint |
| Embeddings | Replaceable embedding provider/model with pinned version |
| API contract | OpenAPI 3.1 |
| Database migrations | Flyway for Java-owned schemas; documented equivalent elsewhere |
| Observability | OpenTelemetry, Prometheus, Grafana, and structured logs; Loki optional |
| Containers | Docker and Docker Compose |
| CI/CD | GitHub Actions with manual deployment initiation initially |
| Infrastructure | Terraform |
| AWS compute | Select ECS/EKS for services and GPU model hosting through an ADR based on learning/cost goals |
| Testing | JUnit/Testcontainers, Angular tests, Python tests, Playwright/Cypress choice via ADR, k6/Gatling choice via ADR |

### 15.1 Model portability

Application services shall call an internal model-access interface using an OpenAI-compatible contract. vLLM is the initial runtime but must be replaceable with OpenAI, Azure OpenAI, Amazon Bedrock through an adapter, or another compatible provider without changing portal APIs or core laboratory domain logic.

### 15.2 Vector-store decision

Phase 2 uses PostgreSQL with pgvector. A separate vector database may be introduced only after documented scale, retrieval, isolation, or operational requirements are demonstrated and an ADR compares alternatives.

---

## 16. Local Developer Experience

### 16.1 Required commands

- `./DevOps/Local/docker-all-up.sh` starts the selected local infrastructure profile in dependency order.
- `./DevOps/Local/docker-all-down.sh` safely stops the local stack without deleting data by default.
- `./DevOps/Local/docker-all-status.sh` reports container state and application-level readiness.
- `./DevOps/Local/docker-all-logs.sh` displays or follows logs with optional component filtering.
- `./DevOps/Local/docker-all-reset.sh` is explicitly destructive, guarded, local-only, and documented.

### 16.2 Script behavior

- Scripts shall work from any current directory by resolving repository root safely.
- Scripts shall support profiles such as `core`, `intelligence`, `observability`, and `all`.
- Up shall wait for readiness, not only container start.
- Status shall show component, container state, readiness, health URL, port, and dependency failures.
- Down shall be idempotent.
- Secrets shall come from ignored local environment files or secret managers, never committed defaults.
- Resource-intensive vLLM shall be optional and shall validate GPU/runtime prerequisites.
- A lightweight compatible-model stub or explicitly configured remote provider may support development without a GPU.
- Application services may run from IDEs while infrastructure runs in containers.

### 16.3 Local infrastructure profiles

`core` initially includes PostgreSQL and Keycloak. `intelligence` adds model runtime and required embedding components. `observability` adds collector, metrics, dashboards, and log tooling. Redis and Kafka are excluded until their implementation phases.

---

## 17. Non-Functional Requirements

### 17.1 Performance

- **NFR-PERF-001:** Non-AI detail reads shall target p95 below 500 ms under the defined demo load in a warmed local/test environment.
- **NFR-PERF-002:** Paginated searches shall target p95 below 1 second for the demo dataset.
- **NFR-PERF-003:** Assistant time to first meaningful streamed event shall target below 2 seconds excluding cold model startup; environment-specific baselines must be recorded.
- **NFR-PERF-004:** Model performance shall record time to first token, inter-token latency, output tokens/second, end-to-end latency, queue time, and concurrent throughput.
- **NFR-PERF-005:** Performance tests shall compare single request and at least 5, 10, and configurable concurrent assistant requests.
- **NFR-PERF-006:** The platform shall define maximum page size, prompt/context size, tool-result size, generated tokens, and upload size.

### 17.2 Availability and resilience

- **NFR-REL-001:** Every service shall expose liveness and readiness endpoints.
- **NFR-REL-002:** Timeouts shall be explicit for every network call.
- **NFR-REL-003:** Retries shall apply only to safe/idempotent operations with exponential backoff and jitter.
- **NFR-REL-004:** Circuit breakers and bulkheads shall protect experience APIs from failed dependencies where justified.
- **NFR-REL-005:** A failed model runtime shall not make authoritative order/specimen/result views unavailable.
- **NFR-REL-006:** A failed knowledge or model dependency shall produce a clear degraded response, not fabricated output.
- **NFR-REL-007:** Asynchronous consumers shall support retry and dead-letter handling with operator visibility.

### 17.3 Scalability

- Stateless application services shall scale horizontally.
- No session-critical state shall reside only in a service instance.
- Cursor or stable offset pagination shall prevent unbounded result reads.
- vLLM parameters such as `max_model_len`, `max_num_seqs`, tensor parallelism, and GPU memory utilization shall be configuration, benchmarked rather than maximized blindly.
- Capacity documentation shall explain the trade-off among model size, context length, concurrency, KV cache, latency, throughput, and cost.

### 17.4 Accessibility and usability

- Portals shall target WCAG 2.2 AA.
- Keyboard navigation, semantic markup, visible focus, adequate contrast, meaningful error messages, and screen-reader labels are required.
- Status shall never be conveyed only by color.
- Dates, units, and numeric reference ranges shall be unambiguous.
- Loading and streaming states shall be cancellable and understandable.

### 17.5 Maintainability

- Capability ownership and dependencies shall be documented.
- Architecture decisions shall be recorded in ADRs.
- Public APIs and events shall be versioned and contract-tested.
- Code coverage thresholds shall focus on domain and safety-critical logic; generated or trivial code may be excluded transparently.
- No circular dependencies between capability modules.
- Shared libraries shall not contain mutable domain entities used by multiple services.

---

## 18. Security, Privacy, and Responsible AI

### 18.1 Security requirements

- Use OAuth 2.0/OIDC and validate issuer, audience, expiry, and scopes/roles.
- Enforce least privilege for database users, services, CI identities, and cloud roles.
- Store passwords, model tokens, signing secrets, and database credentials outside source control.
- Encrypt external traffic; use appropriate in-transit encryption for internal/cloud traffic.
- Use dependency, container, secret, IaC, and static-code scanning in CI.
- Validate inputs and uploads; protect against injection, SSRF, path traversal, unsafe deserialization, and oversized payloads.
- Rate-limit authentication-sensitive and assistant endpoints.
- Record security-relevant actions in tamper-evident audit logs.
- Do not log tokens, passwords, prompt secrets, full customer profiles, or unnecessary result content.

### 18.2 AI threat controls

- Treat retrieved documents and tool output as untrusted content.
- Separate system policy, developer instructions, user input, retrieved context, and tool output.
- Permit only allowlisted tools and validate every tool argument.
- Apply authorization again at each tool endpoint.
- Prevent arbitrary URLs, arbitrary SQL, arbitrary shell commands, and arbitrary internal API invocation.
- Detect and test instructions embedded in documents that attempt to change assistant behavior.
- Redact secrets and sensitive fields before model calls.
- Preserve source provenance through retrieval and answer generation.
- Refuse unsupported medical diagnosis and treatment requests.
- Provide a visible mechanism for users to report an incorrect or unsafe answer.

### 18.3 Demonstration-data privacy

Although all data is synthetic, the design should practice privacy principles: data minimization, role boundaries, secure logging, retention, deletion, and audit. Synthetic records must not accidentally contain real names, email addresses, phone numbers, identifiers, or free text copied from real individuals.

---

## 19. Observability and Auditability

### 19.1 Standard telemetry

Every service shall emit:

- Structured JSON logs.
- OpenTelemetry traces and metrics.
- Service/version/environment metadata.
- Correlation, request, trace, and span identifiers.
- Dependency latency and result status.
- RED metrics: rate, errors, and duration.
- Database pool and query health metrics without sensitive SQL values.

### 19.2 Intelligence telemetry

Assistant execution telemetry shall include:

- Provider, model, and configuration version.
- Prompt-template version, not hidden prompt contents in ordinary logs.
- Retrieval duration, filters, top K, source/chunk identifiers, and scores.
- Tool names, durations, outcomes, and safe result metadata.
- Time to first token, input/output token counts, finish reason, and total latency.
- Safety/refusal result and citation count.
- Evaluation linkage when the call is part of an evaluation run.

### 19.3 Audit

Audit events shall cover sign-in/security events, catalog changes, result release/correction, knowledge publication/retirement, synthetic dataset reset, evaluation promotion decisions, and configuration changes. Audit records shall identify actor, action, target, time, outcome, correlation ID, and before/after metadata where safe.

---

## 20. Testing Strategy

### 20.1 Conventional testing

- Unit tests for domain rules and state transitions.
- Property-based tests for generated timelines and invariants where useful.
- Repository tests with ephemeral PostgreSQL.
- Integration tests using Testcontainers or equivalent.
- OpenAPI request/response and backward-compatibility tests.
- Consumer/provider contract tests for service interactions.
- End-to-end tests for primary portal journeys.
- Accessibility tests plus manual keyboard/screen-reader checks.
- Security and authorization tests, especially cross-customer isolation.
- Performance and load tests.
- Resilience tests for dependency timeout, model failure, retrieval failure, and event duplication.
- Migration tests from an earlier schema version.

### 20.2 AI-specific testing

- Retrieval evaluation with expected source/chunk sets.
- Grounded-answer evaluation with expected and forbidden facts.
- Citation-entailment and citation-presence checks.
- Tool selection, arguments, sequencing, authorization, and failure behavior.
- Prompt injection and data-exfiltration tests.
- Medical-diagnosis refusal tests.
- Cross-customer isolation tests through assistant prompts.
- Model/runtime concurrency benchmarks, including Hugging Face baseline experiments outside the production application and vLLM comparison results.
- Regression reports stored as artifacts, not committed bulky runtime output.

### 20.3 Definition of done for a feature

A feature is done only when:

- Relevant requirement IDs and acceptance criteria are satisfied.
- Authorization and validation are implemented.
- Unit/integration/contract tests pass.
- Observability is present.
- API/event documentation is updated.
- Database migration and rollback/forward strategy are documented when applicable.
- Accessibility is checked for UI changes.
- Threat and AI-safety tests are updated when applicable.
- Local startup and CI remain green.
- No real or proprietary data is introduced.

---

## 21. CI/CD and Environments

### 21.1 GitHub Actions

The monorepo shall use path-aware workflows while retaining full integration validation at appropriate checkpoints.

Pull-request workflows shall:

- Detect affected components.
- Compile/build and run formatting/lint checks.
- Run unit, integration, and contract tests as applicable.
- Validate OpenAPI and event schemas.
- Validate database migrations.
- Run dependency, secret, static-code, container, and IaC scans.
- Build container images without publishing untrusted changes.

Main-branch workflows shall:

- Re-run required checks.
- Produce immutable versioned artifacts and SBOMs.
- Sign/publish approved container images.
- Retain test and evaluation reports.

Deployment workflows shall initially use manual dispatch with explicit environment selection and protected approvals for shared environments. A workflow shall plan infrastructure, deploy in dependency order, run health/smoke tests, and expose rollback instructions/results.

### 21.2 Environments

- `local`: developer machine, synthetic data only.
- `dev`: shared integration environment, synthetic data only.
- `test`: automated functional/performance/evaluation environment, synthetic data only.
- `demo`: portfolio demonstration environment, curated synthetic data only.

No production environment holding real clinical data is part of this project.

---

## 22. AWS Target Architecture Requirements

The detailed AWS design belongs in `REQUIREMENTS-ARCHITECTURE.md` and ADRs. The target must satisfy:

- Private networking for databases and internal services.
- Managed PostgreSQL where practical, with pgvector support verified.
- Container registry and managed container compute.
- Separate CPU application and GPU model-runtime scaling concerns.
- Managed secrets and least-privilege IAM.
- Central logs, metrics, traces, alerts, and cost visibility.
- Encrypted storage, backups, and restore testing.
- Public exposure only through controlled ingress/API layers.
- Terraform-managed infrastructure with separate environment state.
- Budget alarms and an explicit method to stop costly GPU resources.
- A documented low-cost mode that uses a remote compatible provider or disables generative features while retaining deterministic functionality.

The choice among ECS, EKS, EC2 GPU instances, and managed AI providers must be made through ADRs using learning value, operational complexity, availability, and cost—not assumed by code generation.

---

## 23. Product Analytics and Success Metrics

### 23.1 Product metrics

- Percentage of seeded customer journeys completed successfully.
- Median/p95 time for an operations user to locate the first delayed stage.
- Search success rate for known IDs.
- Knowledge publication success/failure rate.
- Synthetic dataset generation duration and validation success.

### 23.2 Intelligence metrics

- Retrieval recall at K against reviewed evaluation examples.
- Citation correctness and coverage.
- Grounded factual correctness.
- Unsupported-claim and hallucination rate.
- Correct refusal rate for diagnosis/treatment prompts.
- Correct authorization behavior for cross-customer prompts.
- Tool-selection and argument correctness.
- Time to first token, end-to-end latency, tokens/second, throughput, and failure rate.
- Regression delta between promoted and candidate configurations.

Numeric release thresholds shall be established after the baseline dataset and model are selected; thresholds must not be invented by a coding agent.

---

## 24. MVP Acceptance Criteria

Phase 1 is accepted when:

1. A new developer can follow `LOCAL-DEVELOPMENT.md`, run the core startup command, and see healthy PostgreSQL, identity, required services, and portals.
2. The repository uses the agreed business-capability structure and documents deployable boundaries.
3. A deterministic synthetic dataset can be generated twice with the same seed and produce an equivalent manifest.
4. A synthetic customer can authenticate and view only their orders and specimen timelines.
5. An operations user can search an order/specimen and see a stable, chronologically ordered investigation timeline.
6. Deterministic delay rules correctly identify seeded delay scenarios and provide the evidence used.
7. A catalog administrator can manage a versioned synthetic test definition and map it to LOINC reference data.
8. OpenAPI contracts, database migrations, authorization tests, integration tests, and critical end-to-end tests pass in CI.
9. Logs, metrics, and traces correlate a portal request through experience and laboratory services.
10. The system visibly declares that all data is synthetic and does not provide medical diagnosis.
11. No Quest branding, proprietary material, real patient data, or employer information is present.
12. AI runtime failure cannot prevent deterministic order/specimen functions, even if an optional experimental model integration exists.

Phase 2 is accepted when published knowledge can be retrieved with valid citations, released synthetic results are displayed correctly, a knowledge-only assistant meets established evaluation baselines, and all Synthea/LOINC imports are versioned and reproducible.

Phase 3 is accepted when an authorized operational investigation uses bounded read-only tools, streams events in deterministic sequence, clearly distinguishes facts from generated explanation, passes safety/authorization tests, and produces complete traces and evaluation records.

Phase 4 is accepted when the platform is reproducibly deployed to AWS from Terraform and manually triggered GitHub Actions, passes smoke/security/performance/evaluation gates, documents cost and teardown, and supports tested rollback or forward recovery.

---

## 25. Delivery Plan and Implementation Order

Coding agents shall implement vertical slices and keep the repository runnable after every slice.

### Milestone 0 — decisions and foundation

- Confirm repository/product name.
- Create architecture principles and ADR template.
- Select build tools and supported framework versions.
- Define ID, timestamp, error, pagination, security, and telemetry conventions.
- Define initial deployable boundaries; prefer a modular start.
- Create local PostgreSQL and identity foundation.
- Establish CI, formatting, dependency management, and secret scanning.

### Milestone 1 — catalog and deterministic synthetic data

- Define project test catalog and workflow rules.
- Implement versioned catalog capability.
- Implement seeded synthetic generation and manifest validation.
- Import a deliberately small LOINC subset with provenance.

### Milestone 2 — order/specimen vertical slice

- Implement order and specimen domain rules, persistence, and APIs.
- Implement event timeline and delay insight rules.
- Build DiagnosticsAdmin search/investigation UI.
- Add integration, contract, E2E, trace, and performance baselines.

### Milestone 3 — customer vertical slice

- Implement customer projection and Synthea ingestion.
- Implement customer authorization boundaries.
- Build DiagnosticServices dashboard, order, and specimen views.

### Milestone 4 — results

- Implement synthetic result generation and result lifecycle.
- Add customer released-result view and administrative result view.
- Verify exact-value preservation and non-diagnostic wording.

### Milestone 5 — knowledge and retrieval

- Author synthetic SOP corpus.
- Implement document versioning, ingestion, chunking, embeddings, pgvector retrieval, citations, and curator UI.
- Establish retrieval evaluation dataset and baseline.

### Milestone 6 — vLLM assistant

- Complete and record standalone Hugging Face versus vLLM learning benchmark.
- Integrate vLLM only through OpenAI-compatible model access.
- Implement knowledge-only assistant, streaming contract, limits, fallbacks, and quality evaluation.

### Milestone 7 — tools and operations intelligence

- Register approved read-only tools.
- Implement deterministic execution plan and ordered streaming.
- Implement delay-investigation answer structure and evidence views.
- Add prompt-injection, authorization, resilience, and concurrency tests.

### Milestone 8 — optional eventing/cache

- Introduce Kafka for documented asynchronous event use cases and Redis for documented cache/session use cases.
- Add outbox, idempotent consumers, replay/dead-letter behavior, and operational dashboards.

### Milestone 9 — AWS delivery

- Define AWS ADRs and Terraform modules.
- Build/publish images and deploy with manually triggered workflows.
- Add cost controls, teardown, smoke tests, evaluation gates, and rollback procedures.

---

## 26. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Too many microservices too early | Start with modular deployables; extract only around proven ownership/scaling needs |
| GPU cost or unavailable local GPU | Optional vLLM profile, compatible stub/remote adapter, explicit AWS budget and shutdown controls |
| AI hallucination | Authoritative tools, citations, evidence-first answer format, refusal behavior, evaluation gates |
| Medical-safety confusion | Strong non-diagnostic boundary, synthetic labels, refusal tests, no treatment recommendations |
| Cross-customer data exposure | Service-enforced authorization, scoped tools, negative tests, minimal logging |
| Public-content licensing problems | Project-owned SOPs/catalog, provenance and terms metadata, no bulk scraping |
| Synthetic data inconsistency | Seeded generation, invariants, manifests, referential/workflow validation |
| Shared database coupling | Schema ownership, separate credentials, API/event access, architecture tests |
| Streaming nondeterminism | Monotonic sequence contract and orchestrator-controlled emission order |
| Model/provider lock-in | OpenAI-compatible internal abstraction and provider adapters |
| Monorepo CI becomes slow | Path-aware CI, caching, affected builds, scheduled full validation |
| Portfolio mistaken for real clinical product | Prominent notices, neutral branding, synthetic identifiers, no compliance claims |

---

## 27. Required Architecture Decisions Before Code Generation

The coding agent must not silently decide the following. Create or update an ADR and obtain product-owner acceptance where the choice materially changes the system:

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

---

## 28. Instructions for Claude Code, Cursor, or Another Coding Agent

When this PRD is supplied to a coding agent:

1. Treat this document as the product source of truth and preserve requirement IDs.
2. Read `REQUIREMENTS-ARCHITECTURE.md`, relevant ADRs, API contracts, and local-development documentation before implementation.
3. Do not generate the entire target tree as empty code or create every planned microservice at once.
4. Propose the next smallest vertical slice, list assumptions and affected requirement IDs, and wait for approval before broad scaffolding.
5. Prefer business capability names in repository structure; keep technical names inside capability implementations.
6. Keep portals behind their corresponding experience APIs.
7. Enforce database and authorization ownership; never solve integration by directly reading another capability's schema.
8. Keep all data synthetic and reject any real/proprietary source material.
9. Implement deterministic facts and rules independently of the LLM.
10. Never allow the model to write authoritative business state.
11. Do not add Kafka, Redis, Kubernetes, or a separate vector database without an accepted requirement and ADR.
12. Pin dependencies and generated artifacts; do not use ambiguous `latest` container tags in reproducible environments.
13. Add tests, telemetry, documentation, and security controls with each feature—not as a final cleanup phase.
14. Never commit credentials, downloaded model weights, large generated datasets, evaluation outputs, or local environment files.
15. Stop and ask when a decision listed in Section 27 has not been made.

Recommended first prompt to the coding agent:

> Read `PRD.md` completely. Do not generate application code yet. Produce a proposed Milestone 0 implementation plan that maps each task to PRD requirement IDs, identifies required ADRs, recommends the smallest initial deployable boundaries, lists open decisions, and defines verification commands. Preserve the business-oriented repository naming. Wait for my approval before creating files.

---

## 29. Open Product Decisions

The following decisions remain intentionally open for the owner:

- Final public product name and GitHub repository description.
- Whether `DiagnosticServices` is the final customer-facing brand name or should become a more user-oriented name while retaining the business capability term internally.
- Exact Phase 1 deployable grouping (recommended: fewer modular applications, not one deployable per final capability).
- The first 20–50 synthetic tests/panels and workflow rules.
- Initial vLLM-served model based on available GPU memory and license.
- AWS monthly budget and whether the learning priority favors ECS, EKS, or a simpler EC2-based GPU runtime.
- Whether the demonstration will be private, temporarily public, or accessed only through recorded screenshots/video.

---

## 30. Source and Attribution Note

This project is inspired by general laboratory diagnostics workflows and the owner's historical domain exposure. Public Quest Diagnostics material supplied during planning confirms only the broad public context that laboratory companies provide test directories, customer result access, and large-scale testing services. It is not a data source for cloning a real product.

All implementation data and documentation must come from the approved sources in Section 12. The project must remain vendor-neutral, use original branding and project-owned identifiers, and clearly state that it is an educational demonstration built with synthetic data.

---

## 31. PRD Change Control

- Material changes to scope, safety boundaries, capability ownership, data sources, or release acceptance criteria require a PRD version update.
- Technical implementation decisions belong in ADRs and must reference the PRD requirements they satisfy.
- Requirement IDs must not be reused for different meanings. Deprecated requirements remain documented with replacement references.
- Generated code is subordinate to the accepted PRD, API contracts, migrations, and ADRs.

