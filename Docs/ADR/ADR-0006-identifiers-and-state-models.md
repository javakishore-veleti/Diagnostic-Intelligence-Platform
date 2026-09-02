# ADR-0006: Domain identifiers and state-transition models

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.6
- **Requirements:** FR-ORD-*, FR-SPC-*, FR-RES-*, PRD §12.4, §13.1

## Context

PRD §13.1 requires opaque identifiers rather than sequential database keys. Separately, laboratory operations depends on identifiers people actually read aloud — an accession number is spoken over a phone. These are two different needs and conflating them produces either unusable operations or a leaky primary key.

Delay and exception detection (FR-INS-*) is derived from state and versioned rules, so the state model is a product surface, not an implementation detail.

## Decision

### Identifiers

- **Primary keys are UUIDv7** — time-ordered, so index locality is preserved without exposing a sequence.
- **API representation is a prefixed opaque string**: `cus_`, `ord_`, `spc_`, `res_`, `tst_`, `knw_`, `cnv_`. The prefix makes a misrouted identifier obvious in a log or a bug report.
- **Human-facing identifiers are separate fields**, never the key: specimen accession `ACC-YYYY-NNNNNN`, catalog test code `LAB-NNNNN`. They are unique, business-formatted, and may be reissued under rules the primary key must never follow.

### State models

**Laboratory order**

`DRAFT → SUBMITTED → IN_PROGRESS → PARTIALLY_RESULTED → COMPLETED`, with `CANCELLED` reachable from any state before `COMPLETED`.

**Specimen**

`EXPECTED → COLLECTED → IN_TRANSIT → RECEIVED → ACCESSIONED → IN_TESTING → TESTED`, with `REJECTED` reachable from `RECEIVED` onward and `STORED` following `TESTED`.

**Result**

`PENDING → PRELIMINARY → FINAL → RELEASED`, with `CORRECTED` reachable from `FINAL` or `RELEASED` and producing a new version rather than mutating the old one.

### Rules that hold across all three

1. Transitions are declared in one table per aggregate and enforced in the domain layer. An illegal transition is a domain error, never a silent no-op.
2. **Every transition emits an operational event** (PRD §12.4) carrying before-state, after-state, actor, occurred-at, and correlation ID. The event stream, not a mutable status column, is the investigation record.
3. **Delay is derived, never stored as opinion.** A specimen is delayed when its time in a state exceeds the threshold for that state in the active rule version. The rule version is recorded with the derived finding so a past investigation stays reproducible after thresholds change.
4. Timestamps are UTC, and *occurred-at* is distinct from *recorded-at* — synthetic data must exercise the gap between them, because real operations does.

## Consequences

- Investigation and turnaround insight fall out of the event stream instead of needing a parallel audit mechanism.
- Corrections are versioned, matching how a laboratory actually issues an amended result.
- More writes per transition. Acceptable at demonstration volumes; revisit if event volume becomes a measured problem.

## Revisit when

A capability needs a state the table cannot express, or derived-delay computation becomes too slow at the target dataset size (PRD §12.3).
