# Current Work

## Work package
- ID: WP-M0-001
- Title: Milestone 0 architecture decision records
- Status: VERIFYING — drafts complete, awaiting owner acceptance
- Owner/agent: Claude Code session
- Started: 2026-09-02
- Last updated: 2026-09-02

## Product linkage
- PRD requirement IDs: PRD §27 decisions 1–15; supports Milestone 0 (PRD §25)
- Milestone: 0 — decisions and foundation
- Relevant ADRs: ADR-0001 through ADR-0015 (all Proposed)

## Intended outcome

Every decision PRD §27 forbids an agent from making silently is drafted as an ADR with options, a recommendation, and consequences — so that later work packages can run without stopping to ask.

## In scope

- Fifteen ADRs, one per PRD §27 decision, status `Proposed`.
- ADR index and template.
- Decision queue recording what still needs the owner.

## Out of scope

- Any application code, build file, or container definition.
- Accepting the ADRs. Only the owner does that.
- The first vertical slice.

## Plan
- [x] Draft ADR-0001..0015 — verification: each maps to one PRD §27 item, states options and consequences
- [x] ADR index with status table
- [x] `DECISION-QUEUE.md` with blocking flags and owner-input items
- [ ] Owner review and acceptance — **blocked on owner**
- [ ] Flip accepted ADRs from `Proposed` to `Accepted`

## Current checkpoint
- Last completed task: fifteen ADRs and the decision queue drafted
- Current branch: `docs/WP-M0-001-milestone-0-adrs`
- Working tree state: committed, not pushed
- Next exact action: owner reviews `Docs/ADR/README.md` and accepts, amends, or rejects each ADR. Nothing downstream may proceed on a `Proposed` ADR.

## Verification evidence

| Command | Result | Time |
|---|---|---|
| Manual review — 15 ADRs against PRD §27's 15 items | 1:1, no gaps | 2026-09-02 |

Note: no automated verification exists yet. Establishing it is WP-M0-002, and it is a prerequisite for any unattended run.

## Decisions and assumptions

- Zero infrastructure budget is treated as a hard constraint, and it shaped ADR-0009 (CPU embeddings), ADR-0011 (Ollama over a rented GPU), ADR-0013 (no Kafka or Redis), and ADR-0014 (defer AWS).
- Development is on Apple Silicon, so vLLM cannot run locally. ADR-0011 addresses this without weakening PRD §15.1.
- ADR statuses stay `Proposed`. Per `CLAUDE.md` §2 no code may depend on an unaccepted ADR.

## Blockers

- **Owner acceptance of ADR-0001 through 0015.** Everything in Milestone 0 depends on it.
- **D-008a** — LOINC licence reading needs human confirmation, not an agent's.
- **D-014a / D-014b** — AWS budget and whether Kubernetes is a goal.
- **D-016** — the first 20–50 synthetic tests, without which the Phase 1 catalog cannot be generated.

## Files changed

- `Docs/ADR/README.md`, `Docs/ADR/ADR-0001..0015-*.md` (new)
- `Docs/Execution/CURRENT-WORK.md`, `Docs/Execution/DECISION-QUEUE.md` (new)
- `CLAUDE.md` — §10.2 autonomy contract for unattended runs
