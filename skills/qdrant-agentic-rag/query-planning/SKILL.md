---
name: qdrant-agentic-rag/query-planning
description: |
  Use when a single query can't answer a complex question, the agent needs to break a question
  into sub-queries, deciding whether to run sub-queries in parallel or sequentially, multi-hop
  questions where the answer to one query informs the next, query decomposition before
  searching Qdrant, or the agent is generating search queries from a user's question.
---

# Query Planning for Agentic RAG

Simple questions map to one query. Complex questions don't. A question like "compare our Q3 revenue from last year against industry benchmarks for SaaS companies in Southeast Asia" needs at least three independent retrievals. An agent that sends this as a single vector query will retrieve whichever fragment has the highest average similarity to all three concepts — which is usually the wrong answer to all three.

## Decomposing complex questions

Use when: the question contains multiple distinct information needs.

- Before searching, prompt the planning LLM to extract a list of atomic sub-questions from the user's query; each sub-question should be answerable with a single retrieval
- For questions with dependency chains (answer B depends on knowing answer A), order sub-queries sequentially and feed prior answers as context into later query embeddings
- For independent sub-questions, run retrievals in parallel against Qdrant to reduce total latency; merge results before assembling context — [qdrant.tech/documentation/concepts/search](https://qdrant.tech/documentation/concepts/search/)
- Include the collection and payload filter scope in the planning step, not just the query text; an agent that searches the wrong collection wastes a round-trip

## Routing queries to collections

Use when: your Qdrant instance has multiple collections for different document types, time ranges, or tenants.

- Maintain a lightweight routing index: embed collection descriptions and retrieve the most relevant collection before sending the actual query
- For time-scoped questions ("last quarter", "since the product launch"), use payload filters to restrict results to the relevant date range rather than routing to a separate collection
- Route by query intent (factual lookup vs. semantic similarity vs. exact match) to select the appropriate vector type: dense for semantic, sparse for exact-term, hybrid for both — [qdrant.tech/documentation/concepts/hybrid-queries](https://qdrant.tech/documentation/concepts/hybrid-queries/)

## Query rewriting

Use when: the user's raw question produces poor retrieval results.

- Rewrite the user question into a declarative form that resembles the documents being searched; questions and document passages embed differently for most models
- For knowledge base search: rewrite "how do I fix X" as "steps to fix X" before embedding
- For evidence retrieval: generate a hypothetical answer to the question (HyDE), embed the hypothetical answer, and search with it; this often finds better matches than embedding the question directly
- Store both the original user query and the rewritten query in the agent's working memory; you need the original for the final response, the rewrite for retrieval

## Tracking query provenance

Use when: the agent's final answer needs to cite sources or be auditable.

- Assign a `query_id` to each sub-query and record it alongside retrieved point IDs; this lets you trace which search produced which context
- Store the full query plan (original question, decomposed sub-queries, rewrites applied) in agent state before any retrieval happens
- Use Qdrant payload's `retrieved_by` or `query_ref` field convention to tag stored memories with the query that produced them — [qdrant.tech/documentation/concepts/payload](https://qdrant.tech/documentation/concepts/payload/)

## What NOT to Do

- Do not send the raw user message as the embedding query without any rewriting; user questions are not written in the same register as indexed document passages
- Do not decompose every question into sub-queries; single-hop questions with sub-query decomposition add latency with no accuracy benefit
- Do not run sequential sub-queries when independent parallel queries are possible; sequential adds per-query LLM latency to every step
- Do not ignore collection routing for multi-collection deployments; wrong-collection retrievals return plausible but incorrect results that are hard to debug
