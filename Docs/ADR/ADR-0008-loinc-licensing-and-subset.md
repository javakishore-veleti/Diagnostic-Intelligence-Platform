# ADR-0008: LOINC licensing, distribution, and imported subset

- **Status:** Proposed — **owner must confirm the licence reading before acceptance**
- **Date:** 2026-09-02
- **PRD decision:** §27.8
- **Requirements:** FR-CAT-*, PRD §12.1, §12.2

## Context

LOINC is the terminology backbone for laboratory results. It is free to use but **not public domain**: it is distributed under a licence that requires registration, carries attribution obligations, and places conditions on redistribution. A public GitHub repository is redistribution.

This is the one Milestone 0 decision with a legal dimension rather than a purely technical one, and it is flagged accordingly.

## Decision

**Do not commit the LOINC release to this repository.** The full table, and any substantial extract of it, stays out of version control regardless of how convenient committing it would be.

What is committed instead:

1. **A curated mapping file** — the project's own test codes (`LAB-NNNNN`) paired with the LOINC codes they map to, covering only the 20–50 tests the synthetic catalog actually defines (PRD §29). This is a small, purposeful extract, not a copy of the database.
2. **An ingestion script** that reads a LOINC release the developer has downloaded themselves after accepting the licence, and loads only the columns the catalog needs — code, long common name, component, property, system, scale, units.
3. **The LOINC release version**, recorded in a manifest exactly as with Synthea (ADR-0007), so a mapping is always traceable to the release it was made against.
4. **The required attribution notice**, displayed in both portals wherever LOINC-derived terminology appears, and stated in the README.

`LOINC_HOME` (or an equivalent documented path) points at the local release. The ingestion step fails with a clear message when it is unset — it never silently proceeds with a partial catalog.

## Consequences

- A new developer must register with Regenstrief and download LOINC once. That friction is the correct trade for licence compliance and is documented in `LOCAL-DEVELOPMENT.md`.
- CI cannot ingest LOINC without a licensed copy; catalog tests therefore run against the committed mapping file and a fixture, not against a live LOINC table.
- If the curated mapping file grows toward being a usable substitute for LOINC itself, this decision has been violated in spirit. Keep it to codes the catalog uses.

## Owner action required

Confirm the current LOINC licence terms permit (a) the curated mapping extract above in a public repository, and (b) the attribution wording chosen. **This ADR should not be accepted on my reading of the licence alone.** If in doubt, the conservative fallback is to publish only project codes and keep the LOINC mapping in an ignored local file.

## Revisit when

The catalog grows substantially, LOINC changes its licence terms, or the project decides to publish a demonstration environment with terminology visible to the public.
