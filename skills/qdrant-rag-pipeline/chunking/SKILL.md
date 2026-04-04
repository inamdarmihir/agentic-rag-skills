---
name: qdrant-rag-pipeline/chunking
description: |
  Use when deciding how to split documents for RAG, choosing chunk size, setting overlap,
  documents are too long to embed whole, retrieved chunks cut off mid-sentence, chunks lack
  context about which document they came from, small chunks return irrelevant fragments,
  large chunks dilute relevance scores, or metadata isn't making it into Qdrant payloads.
---

# Chunking Strategy for Qdrant RAG

The single most common RAG mistake is treating chunking as a preprocessing detail. Chunk boundaries determine what the embedding model sees, what Qdrant stores, and what the LLM gets. Wrong chunks poison every stage downstream.

## Choosing chunk size

Use when: deciding on a fixed character or token count for splits.

- Start with 512 tokens for general-purpose prose; smaller (256) for dense technical reference, larger (1024) for narrative documents where context spans paragraphs
- Match chunk size to your embedding model's effective context window, not its maximum; most models degrade on inputs longer than 512 tokens even if they accept 8192
- Check your model's benchmark results on MTEB for the task type "retrieval" at different input lengths — [qdrant.tech/documentation/embeddings](https://qdrant.tech/documentation/embeddings/)
- Never chunk on token count alone without overlap; a sentence split across two chunks will embed poorly in both

## Overlap

Use when: retrieved chunks are missing the sentence that would have answered the question.

- Set overlap to 10–20% of chunk size as a baseline (e.g., 50–100 tokens for 512-token chunks)
- Increase overlap for documents where key facts often span paragraph boundaries (legal text, research papers)
- Overlap increases storage and index size proportionally; measure before increasing past 25%

## Semantic and structural chunking

Use when: fixed-size splits are cutting through tables, code blocks, or section headers.

- Split on structural boundaries (headings, paragraphs, sentences) before applying size limits
- For code: chunk at function or class boundaries, not mid-function
- For tables: embed the table header with every row chunk as a metadata prefix, not as a separate chunk
- For PDFs: extract and chunk by page then re-split oversized pages; do not embed raw PDF bytes

## Payload design

Use when: retrieved chunks can't be traced back to their source or filtered by document.

- Store `source_url`, `document_id`, `section_title`, `chunk_index`, and `page_number` as payload fields on every point — [qdrant.tech/documentation/concepts/payload](https://qdrant.tech/documentation/concepts/payload/)
- Store the full parent document text in payload under a separate key if you plan to expand retrieved chunks at query time
- Index string payload fields you plan to filter on; unindexed fields scan all points — [qdrant.tech/documentation/concepts/indexing](https://qdrant.tech/documentation/concepts/indexing/)
- Do not put embeddings in payload; Qdrant stores those separately as vectors

## Hierarchical chunking

Use when: small retrieval chunks miss context but large chunks dilute relevance.

- Index both parent chunks (1024 tokens) and child chunks (256 tokens) in the same collection using separate named vectors
- Retrieve using child chunk vectors for precision, then fetch the parent chunk payload for context
- Use `payload` to link child chunks to their parent via a `parent_id` field

## What NOT to Do

- Do not chunk by splitting on `\n\n` alone; blank-line splits produce inconsistent lengths and embed poorly
- Do not embed chunks without any payload; you will not be able to filter, trace, or rerank them
- Do not use the same chunk size for every document type in a mixed corpus
- Do not ignore token counts in favor of character counts; different embedding models tokenize differently
