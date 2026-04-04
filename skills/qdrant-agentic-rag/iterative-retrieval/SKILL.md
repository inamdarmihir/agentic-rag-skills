---
name: qdrant-agentic-rag/iterative-retrieval
description: |
  Use when an agent needs to retrieve multiple times to answer a question, implementing
  retrieval loops, deciding when the agent has enough information to stop searching,
  self-correcting retrieval where the agent retries with a different query, multi-hop
  reasoning that requires successive Qdrant queries, or an agent keeps searching without
  stopping (infinite loop risk).
---

# Iterative Retrieval for Agentic RAG

Iterative retrieval lets the agent refine its search strategy based on what it finds. The risk is uncontrolled loops. An agent with no explicit stopping condition will keep searching until it hits a token budget or timeout. Define termination before implementing iteration.

## Loop termination conditions

Use when: the agent is retrieval-looping without a clear exit.

- Define at least two explicit stopping conditions: (1) a confidence check — the LLM judges retrieved context sufficient to answer the question, and (2) a hard iteration limit (3–5 rounds is typical)
- Emit a structured `retrieval_sufficient: true/false` judgment from the LLM after each retrieval round; this is cheaper than full answer generation and gives you a clean loop exit
- Track which Qdrant point IDs have been retrieved across iterations; do not re-retrieve the same point unless explicitly expanding its context

## Follow-up query generation

Use when: the first retrieval round doesn't fully answer the question.

- After retrieving round N, prompt the agent to identify what information is still missing; use this gap description as the next query, not a restatement of the original question
- Embed the already-retrieved chunks alongside the gap description when generating the next query; this steers the embedding toward unexplored document space
- Use `must_not` payload filters on already-retrieved `document_id` values to exclude documents already seen — [qdrant.tech/documentation/concepts/filtering](https://qdrant.tech/documentation/concepts/filtering/)

## Progressive widening

Use when: early iterations return nothing useful.

- Start with strict filters and high score thresholds; if results are insufficient, loosen filters and lower thresholds in subsequent rounds
- Remove the most restrictive `must` filter condition first; keep tenant isolation and safety filters active regardless of retrieval quality
- Track which filter relaxation produced the useful results; this is signal for improving your initial filter strategy

## Self-correction on failed retrievals

Use when: the agent retrieves results that are irrelevant to the actual question.

- After retrieval, ask the LLM to judge relevance before consuming retrieved content; a binary "does this help answer the question?" judgment costs one forward pass
- On irrelevance judgment: rewrite the query using a different framing, not just different words; "what causes X" vs. "X root cause" vs. "X failure mode" will embed differently
- Switch from dense to hybrid search on the second attempt if the first dense-only retrieval failed; keyword overlap often saves multi-hop chains that fail on semantics alone — [qdrant.tech/documentation/concepts/hybrid-queries](https://qdrant.tech/documentation/concepts/hybrid-queries/)

## Tracking retrieval state

Use when: managing context across multiple retrieval rounds.

- Maintain a running list of retrieved point IDs, scores, and the query that retrieved each; pass this state between iterations
- After each round, summarize retrieved content before the next query; feeding raw chunk text from 3 prior rounds fills context fast
- Store iteration state in Qdrant agent memory when sessions span multiple API calls — see [agent-memory](../agent-memory/SKILL.md)

## What NOT to Do

- Do not implement iterative retrieval without a hard iteration cap; LLMs can generate stop conditions that are never satisfied
- Do not expand context window with every retrieval round without pruning; accumulated chunks from 5 rounds will exceed most LLM context windows
- Do not re-embed the original user question on every iteration; embed the gap description from the prior round
- Do not relax tenant isolation or access control filters during progressive widening; security boundaries are not retrieval quality parameters
