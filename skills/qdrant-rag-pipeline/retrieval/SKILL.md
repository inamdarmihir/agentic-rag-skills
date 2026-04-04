---
name: qdrant-rag-pipeline/retrieval
description: |
  Use when retrieved results are irrelevant, search returns too many or too few results,
  deciding how many results to retrieve for RAG, adding filters to retrieval, using hybrid
  search for RAG, setting a score threshold, query expansion isn't helping, retrieval misses
  obvious matches, or choosing between dense and sparse retrieval for a RAG use case.
---

# Retrieval Strategy for Qdrant RAG

Dense retrieval alone fails on exact-match queries (product codes, names, version numbers). Sparse retrieval alone fails on semantic or paraphrased queries. Most RAG pipelines need both. Start with the failure mode, not the architecture.

## Diagnosing retrieval failures

Use when: answers are wrong and you aren't sure if retrieval or generation is the problem.

- Log the raw top-k results for 20 representative queries before they reach the LLM
- If retrieved chunks are relevant but answers are wrong, the problem is context-assembly, not retrieval
- If retrieved chunks are irrelevant, check score distribution: low max scores mean the query embedding is far from any stored vector (embedding mismatch or missing content), not a retrieval bug

## Dense retrieval tuning

Use when: semantic search isn't finding obvious matches.

- Verify the query is embedded with the same model used to index documents; mismatched models produce random-quality results — [qdrant.tech/documentation/embeddings](https://qdrant.tech/documentation/embeddings/)
- Increase `limit` to 20 and inspect rank positions before cutting back; the correct chunk may exist but rank at position 15
- Switch distance metric to match your embedding model's training: cosine for most sentence transformers, dot product for OpenAI and Cohere — [qdrant.tech/documentation/concepts/search](https://qdrant.tech/documentation/concepts/search/)
- Use `with_payload: true` on search responses so you can inspect which documents are being retrieved

## Hybrid search (dense + sparse)

Use when: queries include product names, IDs, acronyms, or technical terms that dense search misses.

- Add a sparse vector field (BM25 or SPLADE) alongside your dense vector in the collection schema — [qdrant.tech/documentation/concepts/hybrid-queries](https://qdrant.tech/documentation/concepts/hybrid-queries/)
- Use `query_by_interface: prefetch` to run both vector searches in parallel then merge results with Reciprocal Rank Fusion (RRF)
- Set RRF `rank_constant` to 60 as a starting point; lower values weight top results more heavily
- Sparse vectors require a different tokenizer than dense embeddings; do not share the same text preprocessing pipeline

## Payload filtering

Use when: results span multiple tenants, document types, or time periods that shouldn't mix.

- Apply `must` filter conditions before vector search, not after; Qdrant evaluates filters during HNSW traversal — [qdrant.tech/documentation/concepts/filtering](https://qdrant.tech/documentation/concepts/filtering/)
- Index every payload field you filter on; filtering on unindexed fields forces a full scan
- For date range filters, store timestamps as integers (Unix epoch) not strings
- Use `should` filters for optional boosts (e.g., prefer recent documents), not `must`

## Score thresholds

Use when: low-relevance results are making it into the LLM context.

- Set `score_threshold` only after measuring score distributions on representative queries; a threshold that works on test data often fails on production queries
- A minimum score of 0.7 (cosine) works for many sentence transformer models but will be wrong for models using dot product distance
- Prefer filtering by score in post-processing so you can log and tune thresholds without redeploying

## Query expansion and rewriting

Use when: short or ambiguous queries produce poor results.

- Expand queries using an LLM before embedding: "Q3 revenue" becomes "total revenue for Q3, quarterly sales figures, third quarter financial results"
- Use HyDE (Hypothetical Document Embedding): generate a hypothetical answer with an LLM, embed the answer, then search with that embedding — this often outperforms query expansion for factual corpora
- Do not chain query expansion with sparse search without testing; expanded queries degrade BM25 scores by adding noise terms

## What NOT to Do

- Do not set `limit` to 3 without measuring whether the correct chunk is in positions 4–10
- Do not use cosine distance with a model trained with dot product — results will be wrong
- Do not filter on payload fields that aren't indexed in collections over 100k points
- Do not skip score logging in development; you will be tuning blind in production
