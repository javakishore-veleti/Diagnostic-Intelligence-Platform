# ADR-0005: Identity realm, roles, and synthetic user mapping

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.5
- **Requirements:** FR-AUTH-*, PRD §18.1 security, §20.2 cross-customer isolation tests

## Context

Keycloak runs locally; cloud identity must stay replaceable. The personas in PRD §7 need distinct authorization, and cross-customer isolation is a tested requirement — a member must never reach another member's records, including through the assistant.

For unattended work, one property matters above all: **realm configuration must be reproducible without a human clicking through an admin console.**

## Decision

A single realm, `diagnostic-intelligence`, defined as a **versioned JSON export** at `DevOps/Local/Identity/realm-diagnostic-intelligence.json`, imported automatically on container start. No manual Keycloak configuration is ever required, and no configuration exists only in someone's local container.

**Roles** map one-to-one onto PRD §7 personas:

| Role | Persona | Portal |
|---|---|---|
| `member` | Synthetic customer | DiagnosticServices |
| `ops-specialist` | Laboratory operations specialist | DiagnosticsAdmin |
| `catalog-admin` | Laboratory catalog administrator | DiagnosticsAdmin |
| `knowledge-curator` | Knowledge curator | DiagnosticsAdmin |
| `quality-analyst` | Intelligence quality analyst | DiagnosticsAdmin |
| `platform-admin` | Platform administrator | DiagnosticsAdmin |

**Member identity binding.** A `member` token carries a `customer_id` claim holding the opaque customer identifier (ADR-0006). Every member-scoped query filters on the claim, at the service that owns the data. The portal and the prompt are never the enforcement point — this is what makes the isolation tests meaningful.

**Protocol.** Authorization Code with PKCE for both portals; Spring Security resource-server JWT validation in the platform. No client secrets in browser code.

**Synthetic users** are seeded in the realm export: one per operations role, and a small set of member users bound to generated customers. Passwords are fixed, well-known development values, documented as such in `LOCAL-DEVELOPMENT.md`, and used in no other environment.

## Consequences

- A fresh clone reaches a working login with one command and no console work.
- The realm export is a reviewable artifact — a new role or client scope shows up in a diff.
- Keycloak-specific claims stay behind a small token-to-principal mapper so a cloud identity provider can be swapped without touching authorization logic.
- Well-known development passwords are acceptable only because every environment is synthetic. Any real deployment requires a separate ADR.

## Revisit when

A non-local environment is stood up, or a role needs finer-grained (object-level) permissions than a role claim can express.
