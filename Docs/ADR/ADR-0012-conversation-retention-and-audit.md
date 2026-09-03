# ADR-0012: Conversation retention and audit detail

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.12
- **Requirements:** FR-AST-*, PRD §19.2, §19.3, §18.3

## Context

Assistant conversations must be persistent enough to investigate an answer after the fact, and short-lived enough that the system does not accumulate a growing store of interaction data by default. Audit records serve a different purpose and follow different rules.

All data here is synthetic, so this is not a privacy obligation — it is a discipline that keeps the design honest for a context where it would be.

## Decision

**Two stores, two lifetimes.**

**Conversations** live in the `assistant` schema and are retained **30 days**, then purged by a scheduled job. The retention window is configuration, not a hard-coded constant, and the purge job is versioned, tested, and logged.

**Execution records** are retained **indefinitely** and are immutable. Every assistant execution writes:

| Recorded | Why |
|---|---|
| Conversation and turn identifier | Ties the answer to its context |
| Model identifier and version | An answer is not reproducible without it |
| Prompt template version | Prompts change; the answer was produced by one version |
| Retrieval set — chunk IDs, document versions, scores | Makes a citation checkable |
| Tool calls — name, arguments, outcome, duration | Shows what evidence the answer actually rests on |
| Safety policy version | Refusal behaviour is versioned too |
| Token counts and latency breakdown | NFR-PERF-004 |
| Correlation and causation identifiers | Joins the assistant trace to the service traces |

**Never stored:** raw chain-of-thought, hidden prompt content, credentials, or full model context. PRD §19.1 prohibits logging these, and an execution record is a log.

**Audit events** (PRD §19.3) — catalog changes, result release and correction, knowledge publication and retirement, dataset reset, evaluation promotion — are immutable, retained indefinitely, and record actor, action, target, time, outcome, correlation ID, and before/after metadata.

A member may delete their own conversation. Deletion removes the conversation content; **the execution record survives in de-identified form**, because the audit trail is what makes a past answer explicable, and losing it would defeat the platform's central claim.

## Consequences

- An answer given weeks ago can be explained precisely — which sources, which tools, which model.
- Execution records grow without bound. Acceptable at demonstration volume; a partitioning strategy is the eventual answer, not a shorter retention.
- Separating conversation content from execution metadata takes deliberate schema design rather than one wide table.

## Revisit when

Execution-record volume becomes a storage or query problem, or a non-synthetic deployment is ever contemplated — in which case retention becomes a legal question, not an engineering one.
