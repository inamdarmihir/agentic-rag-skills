---
name: qdrant-agentic-rag
description: |
  Use when building an AI agent that uses Qdrant for retrieval, implementing agentic RAG,
  an agent needs to decide what to search for, multi-step retrieval where one answer informs
  the next query, storing agent memory or conversation history in Qdrant, an agent needs to
  plan a retrieval strategy, iterative or self-correcting retrieval loops, or tools that
  search Qdrant as part of an agent's action space.
allowed-tools: [Read, Grep, Glob]
---

# Agentic RAG with Qdrant

Agentic RAG is RAG where the retrieval strategy itself is decided at runtime by a model. The agent chooses what to search for, how many times, and when to stop. The hard problems are not retrieval quality (those are covered in qdrant-rag-pipeline) but query decomposition, loop termination, and memory that persists across turns.

A system that retrieves once is not agentic. A system where the model chooses queries dynamically, evaluates results, and decides whether to search again is.

## Sub-skills

- [query-planning](query-planning/SKILL.md) — decomposing complex questions into targeted sub-queries
- [iterative-retrieval](iterative-retrieval/SKILL.md) — multi-step retrieval loops, termination conditions, self-correction
- [agent-memory](agent-memory/SKILL.md) — storing conversation history, episodic memory, and learned facts in Qdrant
