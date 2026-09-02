# ADR-0010: Assistant orchestration — framework versus thin custom

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.10
- **Requirements:** FR-AST-*, PRD §14, §19.2, §20.2

## Context

The assistant must select from an allowlist of read-only tools, retrieve published knowledge, stream responses with deterministic event ordering, record model, prompt, retrieval, tool, and safety-policy versions for every execution, and produce citations that are traceable to a document version and chunk.

That last set of requirements is unusual. Most orchestration frameworks optimise for getting a working agent quickly; this product's differentiator is the audit trail, which frameworks tend to abstract away precisely where it needs to be explicit.

## Options

- **LangChain / LlamaIndex** — fast to a prototype, large dependency surface, frequent breaking changes, and the execution record must be reconstructed from callbacks rather than owned.
- **Spring AI** — keeps the assistant in the JVM, but the Python ecosystem owns the embedding and evaluation tooling this project needs (ADR-0009).
- **Thin custom orchestration** — a few hundred lines over an OpenAI-compatible client.

## Decision

**Thin custom orchestration, in Python, over the OpenAI-compatible client.** The loop is small and explicit:

```
request → policy check → retrieve (published versions only)
       → assemble prompt (versioned template)
       → model call → validate tool request against allowlist and schema
       → execute tool via owning service (authorization enforced there)
       → assemble response with citations → stream ordered events
       → persist the execution record
```

The reasons are specific rather than ideological:

1. **The execution record is the product.** PRD §19.2 requires every execution to carry its model, prompt, retrieval set, tool calls, and policy versions. Owning the loop makes that a first-class write instead of a callback reconstruction.
2. **Tool validation must sit outside the model** (PRD §14.3). Frameworks that helpfully parse and dispatch tool calls are working against a requirement that the parse be adversarial.
3. **Ordered streaming with monotonic sequence numbers** (PRD §10.12) is a contract, not a convenience. Owning the emitter is the only way to guarantee it.
4. Building it is an explicit learning goal (PRD §5.2), and it is genuinely small — the complexity in this system is in the boundaries, not the loop.

Model access goes through the internal OpenAI-compatible abstraction (PRD §15.1), so the runtime behind it stays replaceable (ADR-0011).

## Consequences

- More code owned, and no framework community to inherit fixes from.
- No breaking framework upgrades, no dependency surface to audit, no abstraction to fight when a requirement is specific.
- Retrieval, chunking, and evaluation still use focused libraries — this decision rejects an *orchestration* framework, not every library.

## Revisit when

The loop grows past roughly a thousand lines, or a genuine multi-agent topology appears that a hand-written loop makes awkward.
