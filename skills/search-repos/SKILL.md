---
name: search-repos
description: Search GitHub for repositories matching a topic or keyword using the gh CLI, rank by stars and maintenance signals, and return a shortlist of candidates. Use when the user asks whether a tool already exists for a given purpose, or wants an overview of popular projects in a space.
---

# Search GitHub Repos

Help the user discover existing GitHub projects that address a stated need, so they can evaluate whether to use, fork, or integrate one instead of building from scratch.

## When to use

Trigger phrases include:
- "Is there already a tool/library/CLI/plugin for X?"
- "Search GitHub for Y"
- "What are the top projects for Z?"
- "Before I build this, check if it already exists"

## Procedure

### 1. Clarify the query if needed

Ask the user at most one short clarifying question only if the target is genuinely ambiguous. Otherwise translate their request into search terms directly.

### 2. Run the search

Use `gh search repos` with sensible defaults. Start broad, then narrow if results are too noisy.

```bash
gh search repos "<keywords>" \
  --sort stars \
  --limit 20 \
  --json name,owner,description,stargazersCount,updatedAt,language,license,url,openIssuesCount,pushedAt
```

Use `--language=<lang>` when the user specified a language. Use topic filters (`--topic=<topic>`) when a canonical topic tag applies (e.g. `--topic=cli`, `--topic=mcp`).

For richer queries (e.g. combining topic with minimum stars), use `gh api`:

```bash
gh api -X GET search/repositories \
  -f q='<keywords> stars:>100 pushed:>2025-01-01' \
  -f sort=stars -f order=desc --jq '.items[] | {name, full_name: .full_name, stars: .stargazers_count, pushed: .pushed_at, language, license: .license.spdx_id, open_issues: .open_issues_count, description, url: .html_url}'
```

### 3. Rank and filter

From the returned set, build a shortlist of the top 5–10 candidates. For each candidate, capture:

- Full name (`owner/repo`)
- Stars
- Last push date (convert to "N months ago" for readability)
- Open issues
- License (flag missing/unclear licenses)
- Primary language
- One-sentence description

Drop candidates that are obviously off-topic (wrong language, archive/mirror, gist-style one-liners) unless the user explicitly wants all results.

### 4. Present results

Output a compact table or list, ordered by stars, but call out any candidates that look abandoned (last push > 12 months ago) with a ⚠️ marker. Offer the user a next step: dive deeper on a specific candidate (hand off to `evaluate-candidates`), broaden/narrow the search, or document findings to `planning/` (hand off to `document-findings`).

### 5. Keep the exact query

Preserve the exact `gh` command you ran. If the user later asks to document the findings, that query must be recorded alongside the research date so results are reproducible.

## Guidance

- **Stars are a rough signal, not the whole story.** A 500-star repo updated last week often beats a 10k-star repo abandoned in 2022.
- **Archived repos** (`--archived=false` by default on newer `gh`) should be excluded unless the user wants historical context.
- **Forks with more stars than the upstream** sometimes indicate the project has moved — note this.
- Do not invent repository names. Only report what the CLI actually returned.
