Here's the updated README with Qdrant branding, badges with versions, and icons throughout:

---

<div align="center">

# Qdrant RAG Skills

**Agent skills for building production RAG and agentic systems with Qdrant**

[![Qdrant](https://img.shields.io/badge/Powered%20by-Qdrant-DC244C?logo=qdrant&logoColor=white)](https://qdrant.tech)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-D97757?logo=anthropic&logoColor=white)](https://code.claude.com)
[![Cursor](https://img.shields.io/badge/Cursor-Compatible-000000?logo=cursor&logoColor=white)](https://cursor.com)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/Skills-12-DC244C)](skills/)
[![qdrant-client](https://img.shields.io/badge/qdrant--client-1.9.0%2B-DC244C?logo=qdrant&logoColor=white)](https://pypi.org/project/qdrant-client/)
[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white)](https://python.org)

Skills encode deep knowledge about RAG pipeline architecture, agentic retrieval patterns, multi-tenancy, sparse vectors, reranking, and payload indexing so coding agents can make the engineering decisions that determine whether your retrieval system actually works in production.

These skills complement the official [qdrant/skills](https://github.com/qdrant/skills) repo, which covers core Qdrant operations (scaling, monitoring, deployment, SDK usage). This repo focuses on application-level patterns: how to build, debug, and scale RAG pipelines and agentic systems on top of Qdrant.

</div>

---

## 📦 Installation

### Claude Code

```bash
/plugin marketplace add inamdarmihir/agentic-rag-skills
```

### Cursor

Add manually via **Settings > Rules > Add Rule > Remote Rule (GitHub)** with `inamdarmihir/agentic-rag-skills`.

### npx skills

```bash
npx skills add https://github.com/inamdarmihir/agentic-rag-skills
```

### Clone / Copy

Clone and copy the `skills/` folder into your agent's skill directory:

| Agent | Skill Directory | Version | Docs |
|-------|----------------|---------|------|
| 🤖 Claude Code | `~/.claude/skills/` | Latest | [docs](https://code.claude.com/docs/en/skills) |
| 🖱️ Cursor | `.cursor/skills/` | 0.50+ | [docs](https://docs.cursor.com/context/skills) |
| 💻 OpenCode | `~/.config/opencode/skill/` | Latest | [docs](https://opencode.ai/docs/skills/) |
| 🧠 OpenAI Codex | `~/.codex/skills/` | Latest | [docs](https://developers.openai.com/codex/skills/) |

---

## ⚡ Quick Start

Skills activate automatically when your question matches their description:

```
"My RAG pipeline returns relevant chunks but the answers are still wrong"
→ qdrant-rag-pipeline/context-assembly activates
  walks through ordering, deduplication, token budgeting

"I need to build an agent that can search Qdrant multiple times to answer a question"
→ qdrant-agentic-rag/iterative-retrieval activates
  covers loop termination, follow-up query generation, progressive widening

"Product codes aren't being found by our vector search"
→ qdrant-sparse-vectors activates
  covers BM25/SPLADE setup and hybrid search with Qdrant

"How do I serve 5000 customers from one Qdrant collection?"
→ qdrant-multitenancy activates
  covers payload-based isolation, tenant_id indexing, shard-per-tenant patterns
```

---

## 🧠 Skills

| Skill | Useful For |
|-------|-----------|
| 🏗️ `qdrant-rag-pipeline` | End-to-end RAG pipeline architecture and debugging |
| ✂️ `qdrant-rag-pipeline/chunking` | Document splitting strategy, chunk size, overlap, metadata |
| 🔍 `qdrant-rag-pipeline/retrieval` | Query strategies, filters, hybrid search, score thresholds |
| 📋 `qdrant-rag-pipeline/context-assembly` | Ordering chunks, deduplication, token budgeting, source attribution |
| 🤖 `qdrant-agentic-rag` | Agentic RAG overview and pattern selection |
| 🗺️ `qdrant-agentic-rag/query-planning` | Complex question decomposition, query rewriting, collection routing |
| 🔁 `qdrant-agentic-rag/iterative-retrieval` | Multi-step retrieval loops, self-correction, termination conditions |
| 💾 `qdrant-agentic-rag/agent-memory` | Conversation history, episodic memory, semantic memory, memory decay |
| 🏢 `qdrant-multitenancy` | Collection-per-tenant vs payload isolation, preventing data leaks |
| 🔤 `qdrant-sparse-vectors` | BM25 and SPLADE sparse vectors, hybrid dense+sparse search |
| 🎯 `qdrant-reranking` | Cross-encoder reranking, candidate set sizing, latency management |
| 🗂️ `qdrant-payload-indexing` | Index types, cardinality, compound filters, debugging slow queries |

---

## 📦 Qdrant Version Compatibility

| Component | Package | Version |
|-----------|---------|---------|
| 🗄️ Vector DB | [Qdrant](https://qdrant.tech) | `1.9.0+` |
| 🐍 Python Client | [qdrant-client](https://pypi.org/project/qdrant-client/) | `^1.9.0` |
| 📦 JS/TS Client | [@qdrant/js-client-rest](https://npmjs.com/package/@qdrant/js-client-rest) | `^1.11.0` |
| 🦀 Rust Client | [qdrant-client](https://crates.io/crates/qdrant-client) | `^1.10.0` |
| 🐹 Go Client | [go-qdrant](https://github.com/qdrant/go-client) | `^1.9.0` |

---

## 🤝 Pair with Official Qdrant Skills

[![qdrant/skills](https://img.shields.io/badge/pairs%20with-qdrant%2Fskills-DC244C?logo=qdrant&logoColor=white)](https://github.com/qdrant/skills)

For best coverage, use these skills alongside the official [qdrant/skills](https://github.com/qdrant/skills) repo:

| If you need... | Use... |
|----------------|--------|
| 🏗️ RAG pipeline architecture | This repo: `qdrant-rag-pipeline` |
| 📈 Collection scaling and sharding | qdrant/skills: `qdrant-scaling` |
| 🤖 Agent memory and iterative retrieval | This repo: `qdrant-agentic-rag` |
| 🔍 Search quality diagnosis | qdrant/skills: `qdrant-search-quality` |
| 🏢 Tenant isolation and multi-tenancy | This repo: `qdrant-multitenancy` |
| 📊 Cluster monitoring | qdrant/skills: `qdrant-monitoring` |
| 🔤 Sparse vectors and hybrid search | This repo: `qdrant-sparse-vectors` |
| 🛠️ SDK setup and code examples | qdrant/skills: `qdrant-clients-sdk` |

---

## 🔌 MCP Servers

Pair with these Qdrant MCP servers for live retrieval context during development:

| Server | Purpose |
|--------|---------|
| 🔍 [mcp-code-snippets](https://github.com/qdrant/mcp-code-snippets) | Search Qdrant docs and code examples across all SDKs |
| 🗄️ [mcp-server-qdrant](https://github.com/qdrant/mcp-server-qdrant) | Store and retrieve memories, manage collections directly |

---

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for skill writing conventions.

Found wrong advice? [Open an issue](../../issues/new) with:

- 📛 The skill name
- 💬 The prompt you gave your agent
- ❌ What the agent said vs ✅ what it should have said

---

<div align="center">
  <sub>Powered by <a href="https://qdrant.tech">Qdrant</a> · Works with <a href="https://code.claude.com">Claude Code</a>, <a href="https://cursor.com">Cursor</a>, <a href="https://opencode.ai">OpenCode</a>, and <a href="https://developers.openai.com/codex">Codex</a></sub>
</div>
