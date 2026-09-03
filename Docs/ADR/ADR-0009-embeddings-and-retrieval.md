# ADR-0009: Embedding model, dimensions, chunking, and retrieval baseline

- **Status:** Proposed — awaiting owner acceptance
- **Date:** 2026-09-02
- **PRD decision:** §27.9
- **Requirements:** FR-KNW-*, FR-AST-*, PRD §15.2, §20.2

## Context

Phase 2 grounds assistant answers in curated SOPs and approved references, with citations traceable to a document version and chunk. Storage is pgvector inside the primary PostgreSQL (PRD §15.2).

Two constraints shape this decision. **Zero infrastructure budget:** embedding must run locally on a developer laptop with no GPU and no paid API. **Reproducibility:** a citation must remain verifiable after the model changes, so the embedding model and dimension are part of the data contract, not a tunable.

## Decision

**Model: `BAAI/bge-small-en-v1.5`, 384 dimensions**, pinned by revision hash and run locally through `sentence-transformers` on CPU.

Why this one:

- 384 dimensions keeps the pgvector index small and HNSW builds fast at demonstration volumes — a real consideration when the whole stack must fit on a laptop.
- Strong retrieval quality for its size on standard benchmarks, and materially better than `all-MiniLM-L6-v2` at the same dimensionality and comparable speed.
- Runs on CPU in reasonable time for a corpus of this scale, and on Apple Silicon in particular. No GPU, no API key, no spend.
- Permissively licensed and freely redistributable.

**Chunking: structure-aware, not fixed-window.** SOPs have headings, and a citation that names a section is worth more than one that names a character offset. Split on heading boundaries first, then subdivide any section exceeding roughly 512 tokens, with about 64 tokens of overlap. Every chunk carries document ID, version, heading path, ordinal, and token count.

**Retrieval baseline:** cosine similarity over an HNSW index, top-k = 8, with a similarity floor below which the result set is treated as empty rather than returned weakly. An empty set produces an explicit "insufficient evidence" answer (PRD §14.4) — it never produces an uncited assertion.

**Versioning.** Model identifier, revision, dimension, and chunking-strategy version are stored on every chunk row. A change to any of them is a **migration plus a full re-embed**, and requires an evaluation comparison against the previous configuration (PRD §20.2). Mixed-version vectors in one index are prohibited.

## Consequences

- Embedding is free and offline. No API dependency in the local loop, and no per-token cost as the corpus grows.
- 384 dimensions is a deliberate ceiling on retrieval quality. If evaluation shows retrieval is the bottleneck, moving to a 768-dimension model is a schema migration, which is why the dimension is recorded per row.
- CPU embedding of a large corpus is slow. Acceptable for curated SOPs; not a route to embedding a large public corpus.

## Revisit when

Evaluation shows retrieval recall is the limiting factor in answer quality, the corpus outgrows CPU embedding, or hybrid (BM25 plus vector) retrieval is needed — PostgreSQL full-text search is already present for that.
