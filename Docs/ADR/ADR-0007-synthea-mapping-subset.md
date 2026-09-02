# ADR-0007: Synthea-to-customer and result mapping subset

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.7
- **Requirements:** FR-DATA-*, PRD §12.1, §12.3

## Context

Synthea generates complete synthetic patient histories as FHIR R4 bundles — encounters, conditions, medications, claims, immunizations, care plans. This platform needs a small slice of that: people, and their laboratory observations. Importing everything would fill the `customer` schema with clinical concepts the product does not model and cannot honestly display.

## Decision

**Import exactly two resource types, and ignore the rest.**

| FHIR resource | Maps to | Fields used |
|---|---|---|
| `Patient` | Customer | Identifier, name, birth date, gender, address (locality only), contact |
| `Observation` (laboratory category, LOINC-coded) | Result — Phase 2 | LOINC code, value, unit, reference range, effective time, status |

Explicitly **not imported**: `Encounter`, `Condition`, `MedicationRequest`, `Claim`, `Immunization`, `CarePlan`, `Procedure`, `AllergyIntolerance`. They model clinical care this product does not provide, and importing them would invite exactly the diagnostic framing PRD §6 prohibits.

**Provenance and reproducibility**

- The Synthea version and the generation seed are recorded in `DataMgmt/DataIngestion/Synthea/manifest.json`, alongside population size and generation date.
- **Generated bundles are not committed.** The manifest plus the generator invocation is enough to reproduce the population byte-for-byte; committing hundreds of megabytes of JSON is prohibited by `CLAUDE.md` §28.14.
- Ingestion is idempotent and keyed on the Synthea patient identifier, so re-running it does not duplicate customers.

**Orders and specimens are not derived from Synthea.** Synthea models results but not laboratory operations — collection, transit, accessioning, rejection, delay. Those come from the project's own generator (PRD §12.1), which produces the operational timeline that a Synthea observation is then attached to. This split is what lets exception scenarios be authored deliberately rather than hoped for.

## Consequences

- The customer record stays deliberately thin: enough to identify a person and authorize access to their orders, nothing more.
- Result values carry real LOINC codes and plausible distributions, which is what makes the catalog mapping exercise meaningful.
- A Synthea version bump can change the population. The manifest makes that visible, and the ingestion tests must catch a schema change rather than silently importing fewer fields.

## Revisit when

Results need clinical context the platform decides to display, or the demonstration needs a population larger than a single generator run produces comfortably.
