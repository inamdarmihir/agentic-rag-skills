---
name: qdrant-agentic-rag/agent-memory
description: |
  Use when storing agent conversation history in Qdrant, implementing long-term agent memory,
  an agent needs to remember facts from prior sessions, episodic memory for agents, semantic
  search over past conversations, Qdrant as an agent's working memory, an agent should recall
  what it learned in a previous interaction, or implementing memory decay or forgetting.
---

# Agent Memory with Qdrant

Qdrant works well as agent memory because it supports both exact-match retrieval (payload filters on session ID, user ID, timestamp) and semantic retrieval (embedding similarity for "what do I know about this topic"). Most memory architectures need both.

The risk is unbounded growth. An agent that writes every turn to Qdrant without any consolidation or pruning accumulates noise faster than signal.

## Memory types and collection design

Use when: deciding how to structure agent memory in Qdrant.

- Separate working memory (current session), episodic memory (past interactions), and semantic memory (distilled facts) into different collections or use payload `memory_type` to distinguish them within one collection
- Working memory: short TTL, high write frequency, filtered by `session_id`; consider using an in-process store and flushing to Qdrant only at session end
- Episodic memory: one point per interaction summary, payload contains `user_id`, `session_id`, `timestamp`, `topics`; embed the summary text for semantic retrieval — [qdrant.tech/documentation/concepts/collections](https://qdrant.tech/documentation/concepts/collections/)
- Semantic memory: distilled facts extracted from conversations, deduplicated, payload contains `confidence`, `source_session_id`, `last_reinforced`

## Writing to memory

Use when: deciding what to store after each agent turn.

- Do not store raw LLM output verbatim; summarize or extract structured facts before embedding and storing
- Write episodic memories at session end, not after every message; within-session context belongs in the prompt, not in Qdrant
- Assign a `memory_type` payload field and an `importance_score` (0.0–1.0) at write time; low-importance memories are pruned first — [qdrant.tech/documentation/concepts/payload](https://qdrant.tech/documentation/concepts/payload/)
- Use upsert with a deterministic point ID derived from content hash to prevent storing duplicate facts across sessions

## Retrieving from memory

Use when: the agent needs to recall prior knowledge before responding.

- At the start of each session, retrieve the top-5 most semantically similar episodic memories to the user's opening message; inject them into the system prompt as context
- Filter by `user_id` before any semantic search; never let one user's memories leak into another user's context
- For factual recall: filter `memory_type = "semantic"` and search by embedding; for chronological recall: filter by `user_id` and sort by `timestamp` payload field
- Use `score_threshold` for memory retrieval; irrelevant memories in the system prompt are worse than no memories

## Memory consolidation

Use when: episodic memory has grown large and retrieval is returning redundant entries.

- Periodically run a consolidation job: cluster recent episodic memories by topic (using Qdrant's own vector search to find near-duplicates), merge clusters into a single semantic memory point, delete the source episodes
- Track `reinforcement_count` in semantic memory payload; facts mentioned across multiple sessions have higher confidence and should be ranked higher
- Set a maximum memory count per user; when exceeded, delete the oldest and lowest-importance points — [qdrant.tech/documentation/concepts/points](https://qdrant.tech/documentation/concepts/points/)

## Memory decay and forgetting

Use when: stale memories are causing the agent to act on outdated information.

- Store `created_at` and `last_accessed` as integer timestamps in payload; use these to implement recency-weighted retrieval
- Decay `importance_score` over time using a background job; old unreinforced memories fall below a pruning threshold naturally
- Hard-delete memories containing PII or user-requested deletions immediately; do not rely on score decay for compliance use cases

## What NOT to Do

- Do not store every message as a separate Qdrant point; embedding and indexing per-message is expensive and produces retrieval noise
- Do not retrieve memory without a `user_id` or `session_id` filter; cross-user memory contamination is a serious privacy failure
- Do not use memory retrieval score as a proxy for confidence in the fact itself; high similarity means the memory matches the query, not that the memory is correct
- Do not skip memory consolidation; unbounded episodic memory degrades retrieval quality and increases latency as the collection grows
