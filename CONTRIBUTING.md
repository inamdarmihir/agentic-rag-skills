# Contributing to Qdrant RAG Skills

Skills encode solutions architect knowledge about RAG pipeline design, agentic retrieval, and Qdrant configuration for AI coding agents. Familiarity with [Qdrant documentation](https://qdrant.tech/documentation/) and the [Agent Skills standard](https://agentskills.io/) is recommended before contributing.

## Structure

```
skills/
  <skill-name>/
    SKILL.md              # skill definition (frontmatter + guidance)
    <sub-skill>/
      SKILL.md            # sub-skill for a specific topic
```

**Skills** (`skills/`): passive knowledge triggered by description matching.

## Writing a skill

### Hub skills (navigation only)

Hub skills are directories containing sub-skills. They provide a framing paragraph and links to sub-skills.

- Declare `allowed-tools: [Read, Grep, Glob]` in frontmatter
- Include `name` and `description` with trigger phrases
- Body is navigation only: title, framing paragraph, links to sub-skills
- Hub description should mention the sub-skill topics so agent routing works

### Leaf skills (actual content)

Leaf skills contain the guidance an agent uses to help users.

- Omit `allowed-tools` from frontmatter
- Description contains `Use when` with 5+ trigger phrases using exact user language
- First paragraph corrects a wrong assumption or forces a diagnostic fork
- Sections named by symptom or scenario, not by feature
- Each section starts with `Use when:` one-liner
- Bullets are imperative with inline doc links at the end
- Ends with `## What NOT to Do` section listing 3–5 anti-patterns
- No code blocks in skills
- Links go to `qdrant.tech/documentation/`, not raw GitHub
- Target 40–80 lines; if over 80, consider splitting into hub + sub-skills

### Content principles

- Lead with a diagnostic fork when the solution depends on context; do not give one answer for all cases
- Mention tradeoffs explicitly; do not present one approach as universally correct
- Use exact user language in descriptions; the agent's routing depends on description matching
- Prefer specific numbers over vague guidance: "512 tokens" beats "medium-sized chunks"
- Cross-link to related skills; skills should form a navigable knowledge graph

## Conventions

### Commit messages

- Lowercase, imperative, no period at end
- Short and direct: `"fix overlap guidance in chunking skill"`, `"add memory decay section"`
- Multi-step changes use bullet points in the body

### PR titles

- Lowercase, technical, under 70 chars
- Action or problem focused: `"add sparse vector setup guidance"`, `"fix incorrect RRF rank constant default"`

### PRs

- Small and focused: one logical change per PR
- 1–2 sentence description of what the PR changes
- Link related issues

## Skill ideas welcome

Gaps not yet covered by this repo or qdrant/skills:

- `qdrant-evaluation` — measuring retrieval quality, building eval sets, NDCG/MRR measurement
- `qdrant-streaming-ingestion` — real-time document ingestion patterns, partial updates
- `qdrant-cold-start` — bootstrapping a new collection with no labeled data
- `qdrant-rag-security` — prompt injection via retrieved content, safe context assembly
