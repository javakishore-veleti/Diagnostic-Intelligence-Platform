# ADR-0011: Initial generative model and GPU requirements

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.11
- **Requirements:** PRD §15.1 model portability, §16.2, NFR-PERF-003/004/005

## Context

The PRD names vLLM as the initial model runtime and requires a documented comparison against a Hugging Face inference baseline (PRD §20.2). It also requires that development work without a GPU (PRD §16.2).

Two facts drive this decision. **vLLM requires CUDA** — it does not run on Apple Silicon, which is the development machine. And the project has **no infrastructure budget**, so a rented GPU running continuously is not available.

Treating these as blockers would stall Phase 3. They are not blockers, because PRD §15.1 already made the model runtime replaceable.

## Decision

**A two-tier arrangement, both tiers OpenAI-compatible so no application code changes between them.**

**Tier 1 — local development default: Ollama.** Ollama runs a small open-weight model on Apple Silicon via Metal and exposes an **OpenAI-compatible `/v1` endpoint**, which is precisely the contract the model-access abstraction targets. Initial model: **Qwen2.5-7B-Instruct** (Apache 2.0, strong instruction-following and tool-use behaviour at 7B, comfortable in laptop memory). `Llama-3.1-8B-Instruct` is an acceptable substitute where its licence is agreeable.

Cost: zero. No GPU rental, no API key, no per-token charge. The assistant is fully developable and testable this way.

**Tier 2 — vLLM, on rented GPU, only when producing the benchmark.** The PRD's vLLM requirement is fundamentally about *measuring* continuous batching and paged attention under concurrency (NFR-PERF-004/005). That is a bounded experiment, not a always-on dependency. It runs on an hourly GPU (RunPod or equivalent), produces the comparison report, and is shut down. The report is committed; the GPU is not kept running.

**Tier 3 — a deterministic stub** for CI: fixed responses, zero latency, no model. Contract and streaming-order tests run against it, so CI needs no model at all.

The `intelligence` Docker profile (PRD §16.3) defaults to Ollama and validates prerequisites before starting. It never silently requires a GPU.

## Consequences

- Phase 3 proceeds at zero cost, with a real model producing real streamed answers.
- The vLLM comparison happens once, deliberately, when there is something to measure — rather than becoming a permanent infrastructure requirement.
- A 7B model is meaningfully weaker than a frontier model at grounded synthesis. This is acceptable and arguably useful: it makes retrieval quality and prompt discipline visible, which a larger model would mask.
- Benchmark numbers from rented hardware must record the exact GPU, driver, and quantisation, or they are not reproducible.

## Revisit when

The benchmark is due, a laptop-sized model proves inadequate for grounded answers under evaluation, or a budget for continuous GPU hosting appears.
