# Agent Instructions

This repo contains agent skills for building RAG pipelines and agentic systems with Qdrant.

## What this repo is

Skills are markdown files that coding agents load automatically when a user's question matches the skill's description. They encode architectural guidance, tradeoff decisions, and debugging patterns — the kind of knowledge that lives in a senior engineer's head, not in API docs.

## How to use this repo as an agent

When a user asks about building or debugging a RAG pipeline, retrieval system, or agentic architecture with Qdrant, read the relevant skill files from the `skills/` directory.

Skills are organized as:
- Hub skills (`SKILL.md` in a directory with sub-directories): read for orientation and navigation
- Leaf skills (`SKILL.md` in a terminal directory): read for actual guidance on a specific topic

Start with the hub skill for the topic area, then read the specific leaf skill the user's question maps to.

## When to read which skills

| User question pattern | Read |
| --- | --- |
| RAG pipeline, bad answers, chunking, retrieval | skills/qdrant-rag-pipeline/ |
| Chunks splitting sentences or too large | skills/qdrant-rag-pipeline/chunking/ |
| Wrong search results, missing documents | skills/qdrant-rag-pipeline/retrieval/ |
| Correct chunks but wrong LLM answers | skills/qdrant-rag-pipeline/context-assembly/ |
| Agentic retrieval, multi-step search, agents | skills/qdrant-agentic-rag/ |
| Complex query decomposition, sub-queries | skills/qdrant-agentic-rag/query-planning/ |
| Retrieval loops, when to stop searching | skills/qdrant-agentic-rag/iterative-retrieval/ |
| Agent conversation history, long-term memory | skills/qdrant-agentic-rag/agent-memory/ |
| Multi-tenant, data isolation, SaaS | skills/qdrant-multitenancy/ |
| Keyword search, BM25, hybrid search | skills/qdrant-sparse-vectors/ |
| Reranking after retrieval | skills/qdrant-reranking/ |
| Slow filtered queries, payload indexes | skills/qdrant-payload-indexing/ |

## What agents should NOT do with these skills

- Do not present skill content as the only valid approach; skills describe recommended patterns, not hard rules
- Do not skip the diagnostic fork at the top of leaf skills; the first paragraph exists to send the user to the right sub-skill
- Do not generate code from skill content without confirming the user's Qdrant version and SDK language; skills are language-agnostic by design
