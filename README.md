# Qdrant RAG Skills - Agent Skills for RAG and Agentic Pipelines

**Agent skills for building production RAG and agentic systems with Qdrant**

Skills encode deep knowledge about RAG pipeline architecture, agentic retrieval patterns, multi-tenancy, sparse vectors, reranking, and payload indexing so coding agents can make the engineering decisions that determine whether your retrieval system actually works in production.

These skills complement the official [qdrant/skills](https://github.com/qdrant/skills) repo, which covers core Qdrant operations (scaling, monitoring, deployment, SDK usage). This repo focuses on application-level patterns: how to build, debug, and scale RAG pipelines and agentic systems on top of Qdrant.

## Installation

### Claude Code

```
/plugin marketplace add <your-github-username>/qdrant-rag-skills
```

### Cursor

Add manually via **Settings > Rules > Add Rule > Remote Rule (GitHub)** with `<your-github-username>/qdrant-rag-skills`.

### npx skills

```
npx skills add https://github.com/<your-github-username>/qdrant-rag-skills
```

### Clone / Copy

Clone and copy the `skills/` folder into your agent's skill directory:

| Agent | Skill Directory | Docs |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | [docs](https://code.claude.com/docs/en/skills) |
| Cursor | `.cursor/skills/` | [docs](https://docs.cursor.com/context/skills) |
| OpenCode | `~/.config/opencode/skill/` | [docs](https://opencode.ai/docs/skills/) |
| OpenAI Codex | `~/.codex/skills/` | [docs](https://developers.openai.com/codex/skills/) |

## Quick Start

Skills activate automatically when your question matches their description.

```
"My RAG pipeline returns relevant chunks but the answers are still wrong"
→ qdrant-rag-pipeline/context-assembly activates, walks through ordering, deduplication, token budgeting

"I need to build an agent that can search Qdrant multiple times to answer a question"
→ qdrant-agentic-rag/iterative-retrieval activates, covers loop termination, follow-up query generation, progressive widening

"Product codes aren't being found by our vector search"
→ qdrant-sparse-vectors activates, covers BM25/SPLADE setup and hybrid search with Qdrant

"How do I serve 5000 customers from one Qdrant collection?"
→ qdrant-multitenancy activates, covers payload-based isolation, tenant_id indexing, shard-per-tenant patterns
```

## Skills

| Skill | Useful for |
| --- | --- |
| qdrant-rag-pipeline | End-to-end RAG pipeline architecture and debugging |
| qdrant-rag-pipeline/chunking | Document splitting strategy, chunk size, overlap, metadata |
| qdrant-rag-pipeline/retrieval | Query strategies, filters, hybrid search, score thresholds |
| qdrant-rag-pipeline/context-assembly | Ordering chunks, deduplication, token budgeting, source attribution |
| qdrant-agentic-rag | Agentic RAG overview and pattern selection |
| qdrant-agentic-rag/query-planning | Complex question decomposition, query rewriting, collection routing |
| qdrant-agentic-rag/iterative-retrieval | Multi-step retrieval loops, self-correction, termination conditions |
| qdrant-agentic-rag/agent-memory | Conversation history, episodic memory, semantic memory, memory decay |
| qdrant-multitenancy | Collection-per-tenant vs payload isolation, preventing data leaks |
| qdrant-sparse-vectors | BM25 and SPLADE sparse vectors, hybrid dense+sparse search |
| qdrant-reranking | Cross-encoder reranking, candidate set sizing, latency management |
| qdrant-payload-indexing | Index types, cardinality, compound filters, debugging slow queries |

## Pair with Official Qdrant Skills

For best coverage, use these skills alongside the official [qdrant/skills](https://github.com/qdrant/skills) repo:

| If you need... | Use... |
| --- | --- |
| RAG pipeline architecture | This repo: `qdrant-rag-pipeline` |
| Collection scaling and sharding | qdrant/skills: `qdrant-scaling` |
| Agent memory and iterative retrieval | This repo: `qdrant-agentic-rag` |
| Search quality diagnosis | qdrant/skills: `qdrant-search-quality` |
| Tenant isolation and multi-tenancy | This repo: `qdrant-multitenancy` |
| Cluster monitoring | qdrant/skills: `qdrant-monitoring` |
| Sparse vectors and hybrid search | This repo: `qdrant-sparse-vectors` |
| SDK setup and code examples | qdrant/skills: `qdrant-clients-sdk` |

## MCP Servers

Pair with these Qdrant MCP servers for live retrieval context during development:

| Server | Purpose |
| --- | --- |
| [mcp-code-snippets](https://github.com/qdrant/mcp-code-snippets) | Search Qdrant docs and code examples across all SDKs |
| [mcp-server-qdrant](https://github.com/qdrant/mcp-server-qdrant) | Store and retrieve memories, manage collections directly |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for skill writing conventions.

Found wrong advice? [Open an issue](../../issues/new) with:
- The skill name
- The prompt you gave your agent
- What the agent said vs what it should have said

## License

Apache 2.0 — see [LICENSE](LICENSE)
