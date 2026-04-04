---
name: qdrant-sparse-vectors
description: |
  Use when adding keyword search to Qdrant, implementing BM25 or SPLADE with Qdrant,
  hybrid search combining dense and sparse vectors, exact term matching in Qdrant,
  product codes or identifiers aren't being found by vector search, deciding between
  BM25 and SPLADE sparse encoders, setting up a sparse vector collection, or
  debugging sparse vector search quality.
---

# Sparse Vectors in Qdrant

Dense vectors capture semantic meaning. Sparse vectors capture lexical presence. Queries with exact terms, product codes, names, and abbreviations score poorly on dense search but score correctly on sparse. Adding sparse vectors to a dense-only collection is usually the highest-leverage retrieval improvement for technical or product corpora.

## When to add sparse vectors

Use when: deciding if sparse vectors are worth the extra storage.

- Add sparse vectors when your query set includes: named entities, product SKUs, version strings, acronyms, or domain jargon that your embedding model wasn't trained on
- Run a diagnostic first: take 20 queries that produce bad results, check if the correct document contains the exact query terms; if yes, it's a lexical matching problem that sparse vectors fix
- Do not add sparse vectors to solve a retrieval problem caused by bad chunking or mismatched embedding models; fix those first

## Sparse vector collection setup

Use when: adding sparse vectors to a new or existing Qdrant collection.

- Define a sparse vector configuration alongside your dense vector in the collection schema using `sparse_vectors` parameter — [qdrant.tech/documentation/concepts/vectors](https://qdrant.tech/documentation/concepts/vectors/)
- Name the sparse vector field consistently: `sparse`, `bm25`, or `splade` based on the encoder; you'll reference this name in every query
- Sparse vectors in Qdrant use the inverted index, not HNSW; they are stored and searched differently from dense vectors, with no separate index creation step required
- Set `modifier: idf` on the sparse vector config when using BM25-style term weighting for document frequency normalization

## BM25 vs SPLADE

Use when: choosing a sparse encoding approach.

- Use **BM25** when: you need a deterministic, interpretable score, your documents are in a well-resourced language, and you want tokenizer-level control over what counts as a term
- Use **SPLADE** when: you want sparse vectors that generalize beyond exact terms (SPLADE learns to expand queries with related tokens), you have a GPU for encoding, or your corpus has paraphrasing and synonyms that BM25 misses
- SPLADE requires running a transformer model at index and query time; BM25 only needs a tokenizer; the operational cost difference is significant in production
- Both produce sparse vectors in Qdrant's `{index: weight}` format; the collection schema is identical

## Hybrid search with Qdrant

Use when: combining dense and sparse results.

- Use `prefetch` queries to run dense and sparse searches in parallel, then merge with Reciprocal Rank Fusion (RRF) — [qdrant.tech/documentation/concepts/hybrid-queries](https://qdrant.tech/documentation/concepts/hybrid-queries/)
- Set RRF `rank_constant` to 60 as a starting baseline; decrease to 20 if you want top-ranked results to dominate; increase to 100 to weight lower-ranked results more
- Retrieve a larger candidate set from each searcher (e.g., top-50 dense, top-50 sparse) before fusion; RRF with small candidate sets loses precision
- Do not average raw scores from dense and sparse searches; cosine distance and BM25 scores are on incomparable scales

## Tokenization consistency

Use when: sparse vector retrieval returns unexpected misses.

- Use the same tokenizer at index time and query time; a mismatch (e.g., whitespace tokenizer at index, WordPiece at query) will produce zero overlap on identical text
- Lowercase and normalize text before tokenization unless your use case requires case-sensitive matching
- For multilingual corpora, verify your tokenizer handles the languages present; BM25 tokenizers are language-specific

## What NOT to Do

- Do not index sparse vectors without payload; you will not be able to filter sparse results by tenant, document type, or date
- Do not use sparse-only search for semantic queries; sparse vectors have zero score for queries where none of the query terms appear in the document
- Do not tune dense and sparse retrievers independently in isolation; they interact at fusion time and need to be evaluated together
- Do not use raw TF-IDF weights as sparse vectors without IDF normalization; high-frequency terms will dominate scores regardless of relevance
