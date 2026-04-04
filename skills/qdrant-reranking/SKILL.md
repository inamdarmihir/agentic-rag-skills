---
name: qdrant-reranking
description: |
  Use when retrieved results are relevant but in the wrong order, adding a reranker after
  Qdrant retrieval, deciding between a cross-encoder reranker and vector score ordering,
  Cohere Rerank or BGE reranker with Qdrant, retrieval returns correct documents but the
  most relevant one is ranked 5th, reranking latency is too high, or deciding how many
  candidates to retrieve before reranking.
---

# Reranking Retrieved Results from Qdrant

Vector similarity scores are a proxy for relevance, not a measurement of it. A chunk with cosine score 0.91 is not necessarily more relevant to the query than one with 0.87. Cross-encoder rerankers measure relevance directly by comparing the query and document together, but they're too slow to run on the full collection. The standard pattern is retrieve-many, rerank-few.

## When reranking helps

Use when: diagnosing whether reranking is the right fix.

- Reranking helps when: the correct document is in the top-20 results but not the top-5, query and document phrasings differ but are semantically equivalent, or answer quality varies across query formulations
- Reranking does not help when: the correct document isn't retrieved at all (increase `limit` or fix the retrieval strategy first), or the collection is too small for rank order to matter (<1k points)
- Measure NDCG@5 and NDCG@10 before adding a reranker; if they're similar, reranking won't help

## Retrieve-many, rerank-few

Use when: setting up a reranking pipeline with Qdrant.

- Retrieve 20–50 candidates from Qdrant using vector search, then rerank to select top-5 for context — [qdrant.tech/documentation/concepts/search](https://qdrant.tech/documentation/concepts/search/)
- Use `with_payload: true` to return chunk text in the search response; the reranker needs the text, not just the vector
- Pass both the query string and each retrieved chunk's text to the reranker as (query, passage) pairs
- Select the top-N from the reranker's output for LLM context assembly; discard the rest regardless of their original vector score

## Cross-encoder rerankers

Use when: choosing a reranker model.

- **Cohere Rerank**: strong multilingual support, API-based, adds a network call; use when latency budget allows and language coverage matters
- **BGE reranker (BAAI/bge-reranker-v2)**: runs locally, comparable quality to Cohere on English, requires a GPU for reasonable latency; use for on-premises or privacy-sensitive deployments
- **ms-marco-MiniLM cross-encoders**: smallest and fastest, good for high-throughput pipelines where latency is tighter than accuracy
- All cross-encoders require running (query, passage) pairs sequentially or in small batches; batch size 8–16 balances GPU utilization and latency

## Latency management

Use when: reranking latency is unacceptable in production.

- Rerank only the top-20 candidates, not all retrieved results; going from top-50 to top-20 cuts reranker calls by 60% with minimal quality loss
- Cache reranker results keyed by (query_hash, point_id); repeated queries with the same top candidates avoid redundant reranker calls
- For real-time use cases, run the Qdrant query and the reranker in parallel when possible: start reranking the first batch of results while Qdrant is still returning the rest
- If cross-encoder latency is too high, a bi-encoder reranker (same architecture as the retriever) can improve ordering at a fraction of the cost

## Combining reranking with Qdrant's built-in fusion

Use when: using hybrid search and reranking together.

- Apply RRF fusion on dense and sparse results first inside Qdrant, then rerank the fused top-N externally — [qdrant.tech/documentation/concepts/hybrid-queries](https://qdrant.tech/documentation/concepts/hybrid-queries/)
- Do not rerank before RRF fusion; the reranker expects candidates to already be post-fusion
- Retrieve a larger candidate set for hybrid + rerank pipelines (50–100 candidates) because both fusion and reranking benefit from more initial candidates

## Evaluating reranker quality

Use when: deciding if the reranker is actually improving results.

- Compare NDCG@5 with and without the reranker on a labeled evaluation set of at least 100 query-answer pairs
- Check reranker latency at your 95th percentile query load; median latency is misleading for variable-length document reranking
- Run ablations: test vector order vs. reranked order on the same retrieved candidates; if they're nearly identical, the reranker is adding latency without value

## What NOT to Do

- Do not rerank before verifying the correct document is in the initial candidate set; a reranker can only reorder what retrieval provides
- Do not use reranker scores as absolute relevance thresholds; they are relative scores across a candidate set, not absolute quality judgments
- Do not skip caching for high-traffic queries; reranker API calls at scale add up fast
- Do not run a cross-encoder reranker on 200+ candidates in a synchronous request path without measuring p99 latency
