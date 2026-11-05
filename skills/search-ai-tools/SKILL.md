---
name: search-ai-tools
description: Search GitHub for AI/ML-related tools with heavy weighting on recency, since stale AI projects (unmaintained 6+ months) are usually worthless given how fast the ecosystem moves. Use when the user is looking for AI libraries, MCP servers, LLM tooling, agents, prompt frameworks, or ML utilities.
---

# Search GitHub for AI Tools

AI/ML tooling moves fast. A library that was best-in-class twelve months ago is often broken against current model APIs, SDK versions, or provider behaviour today. This skill applies a stricter recency filter than the general `search-repos` skill.

## When to use

Trigger phrases include:
- "Find an AI/ML library for X"
- "Is there an MCP server for Y?"
- "What's a good LLM framework for Z?"
- "Search for agent frameworks / prompt management / eval tools / RAG libraries"

## Procedure

### 1. Frame the search with a recency cutoff

Default cutoff: **last push within the last 6 months**. Unless the user overrides, exclude anything staler than that at the query level.

```bash
# Compute a 6-month-ago date
CUTOFF=$(date -d '6 months ago' +%Y-%m-%d)

gh api -X GET search/repositories \
  -f q="<ai-keywords> pushed:>$CUTOFF" \
  -f sort=stars -f order=desc --jq '.items[] | {
    full_name, stars: .stargazers_count, pushed_at, updated_at,
    language, license: .license.spdx_id,
    open_issues: .open_issues_count,
    description, url: .html_url
  }'
```

Useful query operators to combine:
- `topic:llm`, `topic:agents`, `topic:rag`, `topic:mcp`, `topic:prompt-engineering`
- `stars:>50` to strip hobby projects
- `archived:false`

### 2. Tier the results

Sort candidates into three tiers based on recency, then by stars within each tier:

- **Active** — pushed within last 30 days
- **Maintained** — pushed 1–6 months ago
- **Stale (⚠️)** — pushed 6–12 months ago; include only if the user explicitly asked for a complete picture
- **Dead** — pushed > 12 months ago; exclude entirely unless specifically requested

### 3. Look for activity signals beyond last-push

- **Recent releases** — `gh api repos/<owner>/<repo>/releases --jq '.[0].published_at'`
- **Commit cadence** — `gh api repos/<owner>/<repo>/commits --jq '[.[].commit.author.date] | .[:5]'`
- **Open PRs** — healthy projects have a steady stream, not hundreds untouched

Flag projects where last-push is recent but commits are only dependency bumps (Dependabot churn) — that's not real maintenance.

### 4. Present results

Output grouped by tier. For each candidate include:

- Full name, stars, last push ("N days/weeks/months ago")
- Language, license, open-issue count
- One-sentence summary of what it does
- A short note on maintenance signal ("active releases", "only Dependabot commits", "solo maintainer", etc.) where visible

### 5. Offer next steps

- Deeper evaluation via `evaluate-candidates`
- Documentation via `document-findings` (AI research especially benefits from date stamping — what's good today may be obsolete in 3 months)

## Guidance

- **A 2k-star AI project last updated 14 months ago is almost always worse than a 200-star project updated yesterday.** Say so explicitly.
- **Watch for abandoned demos** — viral projects from blog posts or tweet threads often look popular but have no ongoing maintenance.
- **LLM API compatibility rots fast.** Anything predating the current model generations may be broken.
- Do not invent projects. Report only what the CLI returned.
