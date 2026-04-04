---
name: qdrant-rag-pipeline/context-assembly
description: |
  Use when deciding how to pack retrieved chunks into an LLM prompt, retrieved results are
  relevant but the LLM still gives wrong answers, duplicate content is reaching the LLM,
  context window is being exceeded by retrieved chunks, ordering retrieved chunks in the
  prompt, deciding how many chunks to include, or LLM ignores parts of the context.
---

# Context Assembly for Qdrant RAG

Retrieval quality and generation quality are independent. Relevant chunks assembled badly produce worse answers than fewer but well-assembled chunks. The two most common assembly bugs are duplicate content (same fact from two overlapping chunks) and lost-in-the-middle (the correct chunk is retrieved but placed where the LLM ignores it).

## Deduplication

Use when: the same fact appears multiple times in the LLM context.

- After retrieval, group chunks by `document_id` payload field; keep the highest-scoring chunk per document for the same passage
- For overlapping chunks (set during indexing), compare chunk text with a character-level similarity threshold (>0.8 overlap = duplicate); remove the lower-scoring one
- Do not rely on vector similarity alone to deduplicate; near-duplicate chunks can have very different embedding distances

## Ordering chunks in the prompt

Use when: the LLM is ignoring content that was retrieved correctly.

- LLMs attend most strongly to content at the beginning and end of the context window (primacy and recency effects); put the highest-relevance chunk first and the second-highest chunk last
- For multi-hop questions where chunk B answers a follow-up to chunk A, order chunks in logical dependency order regardless of score rank
- When chunks span multiple documents, group by document so the LLM sees contiguous source passages rather than interleaved fragments

## Context window budgeting

Use when: retrieved chunks exceed the LLM's context window.

- Measure token count of assembled context before every LLM call; do not assume `limit=5` chunks will fit
- Reserve at minimum 25% of the context window for the system prompt and generated answer
- If assembled chunks exceed budget: first drop the lowest-scoring chunk, then truncate the longest chunk from its end (not middle), then reduce `limit` in the retrieval query
- Store chunk token length in payload at index time so you can budget without re-tokenizing at query time

## Chunk expansion

Use when: retrieved short chunks lack enough surrounding context for the LLM to answer.

- At index time, store the full parent passage in payload under `parent_text` alongside the child chunk
- At query time, retrieve using the child chunk's embedding for precision, then swap in `parent_text` for the LLM context
- Expand only chunks whose score is above your relevance threshold; expanding low-scoring chunks injects noise

## Adding source attribution

Use when: the LLM needs to cite sources or the user needs to verify answers.

- Prefix each chunk in the prompt with its `source_url` and `section_title` from payload
- Number chunks sequentially in the prompt and instruct the LLM to cite by number
- Store `chunk_index` and `document_id` in payload so retrieved points can be linked back to the original document — [qdrant.tech/documentation/concepts/payload](https://qdrant.tech/documentation/concepts/payload/)

## Reranking before assembly

Use when: the top retrieval results by vector score aren't the most relevant for the specific query.

- Apply a cross-encoder reranker (e.g., Cohere Rerank, BGE reranker) to the top-20 retrieved chunks before selecting the final top-5 for context
- Retrieve a larger candidate set (`limit=20`) from Qdrant, rerank externally, then pack the top-N reranked results; this consistently outperforms retrieving top-5 directly
- See [qdrant-reranking](../../qdrant-reranking/SKILL.md) for reranking patterns

## What NOT to Do

- Do not concatenate all retrieved chunks into a single string without separators; the LLM cannot tell where one chunk ends and another begins
- Do not always put chunks in retrieval score order; score rank and answer relevance diverge for multi-part questions
- Do not skip token counting before submitting to the LLM; silent truncation produces answers that ignore key context
- Do not include every retrieved chunk regardless of score; a low-scoring chunk that contradicts the high-scoring ones degrades answer quality
