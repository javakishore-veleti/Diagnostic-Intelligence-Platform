# ADR-0014: AWS container and model-runtime hosting pattern

- **Status:** Proposed — **deferred to Phase 4**, recorded now to prevent premature commitment
- **Date:** 2026-09-02
- **PRD decision:** §27.14
- **Requirements:** PRD §22, §8.4, §29 (budget is an open owner decision)

## Context

PRD §22 requires an AWS target architecture, and §27.14 asks for an ECS/EKS choice "based on learning and cost goals". PRD §29 leaves the monthly budget explicitly open.

The project currently has **no infrastructure budget**, and Phase 4 is four phases away. Committing now to a hosting pattern would be deciding against unknowns — but leaving it entirely open invites Phase 1 code that assumes a platform it will not get.

## Decision

**Defer the commitment; record the default so nothing is built against a contradictory assumption.**

Recorded default, to be confirmed or replaced when Phase 4 begins:

| Concern | Default | Reasoning |
|---|---|---|
| Container hosting | **ECS on Fargate** | No cluster to manage or pay for idle; scales to a small floor; matches a small number of deployables (ADR-0001) |
| Database | **RDS PostgreSQL**, smallest supported instance, or Aurora Serverless v2 at minimum capacity | pgvector is available on both; no separate vector store to run (ADR-0009) |
| Identity | Keycloak on ECS, or a managed provider if the cost comparison favours it | Kept behind the token mapper from ADR-0005 |
| Model runtime | **No GPU in AWS.** vLLM benchmarking on hourly rented GPU (ADR-0011); the deployed assistant uses a configured remote OpenAI-compatible provider or is disabled in the demonstration environment | GPU instances are the single largest cost in this architecture, by a wide margin |
| Environments | One (`demo`). `dev` and `test` stay local | Three always-on environments is three times the floor cost for a solo project |

**EKS is rejected** unless Kubernetes itself becomes a learning goal the owner states explicitly. It adds a control-plane charge and substantial operational surface for a workload that does not need it.

**Constraints Phase 1–3 must respect** so this decision stays open:

- Configuration comes from the environment; nothing assumes a specific secret store, service discovery mechanism, or filesystem layout.
- No dependency on a Kubernetes primitive.
- Containers are twelve-factor: stateless, logging to stdout, healthcheck endpoints, graceful shutdown.

## Owner action required

Two figures decide this, and only the owner has them: a **monthly budget ceiling**, and whether **Kubernetes experience is itself a goal**. Without the first, no honest hosting recommendation is possible; the second is the only argument for EKS here.

## Consequences

- Phase 1–3 work cannot accidentally bind the project to a platform.
- The estimated floor for the recorded default is small but not zero — Fargate tasks, RDS, and a load balancer each carry a monthly cost even when idle. A precise estimate belongs with the Phase 4 work package, not this ADR.

## Revisit when

Phase 4 begins, or the owner sets a budget.
