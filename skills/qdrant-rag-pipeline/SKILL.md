---
name: qdrant-rag-pipeline
description: |
  Use when building a RAG pipeline with Qdrant, setting up retrieval-augmented generation,
  debugging bad RAG answers, deciding how to chunk documents, choosing a retrieval strategy,
  assembling context for an LLM, improving RAG answer quality, indexing documents into Qdrant,
  or diagnosing why retrieved chunks aren't relevant.
allowed-tools: [Read, Grep, Glob]
---

# RAG Pipeline with Qdrant

A RAG pipeline has three independently tunable stages. Bad answers almost always trace back to exactly one of them: the chunks fed in, the retrieval logic pulling them out, or the context window assembled for the LLM. Fixing the wrong stage wastes time.

Diagnose first: are retrieved chunks relevant but answers still wrong (context-assembly problem), are chunks irrelevant (retrieval problem), or are chunks retrieved correctly but lack the content needed to answer (chunking problem)?

## Sub-skills

- [chunking](chunking/SKILL.md) — document segmentation strategies, chunk size, overlap, metadata
- [retrieval](retrieval/SKILL.md) — query strategies, filters, hybrid search, score thresholds
- [context-assembly](context-assembly/SKILL.md) — ranking retrieved chunks, deduplication, prompt packing
