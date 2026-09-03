# ADR-0003: Angular workspace and shared UI library

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.3
- **Requirements:** FR-CXP-*, FR-ADM-*, PRD §17.4 accessibility

## Context

Two portals with different audiences and authorization models, sharing a visual language and a small set of components — most notably the synthetic-data notice that PRD §6 requires on every portal.

## Options

- **Two separate repositories or workspaces** — duplicate tooling and shared code by copy-paste.
- **Nx monorepo** — affected-project detection and generators, at the cost of a second build system to learn and keep current.
- **One Angular CLI workspace** — two applications plus a library, using only what Angular ships.

## Decision

A single **Angular CLI workspace** under `Portals/`:

```
Portals/
├── angular.json
├── projects/
│   ├── diagnostic-services/     application — member portal
│   ├── diagnostics-admin/       application — operations portal
│   └── shared-experience/       library — @dip/shared-experience
```

Conventions:

- **Standalone components and signals** throughout. No NgModules in new code.
- The shared library holds only what both portals genuinely use: the synthetic-data banner, the problem-details error renderer (ADR-0004), correlation-ID display, the data table, and the design tokens. Feature code never goes there — the same rule `CLAUDE.md` §5.4 applies to `SharedServices`.
- Each application talks to exactly one experience API through a **generated client** (ADR-0004). Portals never call a laboratory capability.
- Accessibility is enforced in CI with axe-core component tests, not left to a manual pass.

Nx is deliberately not adopted. Its main benefit — affected-project detection — is replaceable in this workspace with a path filter in GitHub Actions, and a second build system is a second thing to keep current.

## Consequences

- One `npm install`, one lockfile, one Angular version to upgrade.
- Two applications share a version number; they are released together, which is correct while both are served by one repository and one owner.
- If a third application ever appears, revisit Nx rather than growing bespoke scripts.

## Revisit when

A third application is added, workspace-wide builds become slow enough to discourage running them, or the two portals need independent release cadences.
