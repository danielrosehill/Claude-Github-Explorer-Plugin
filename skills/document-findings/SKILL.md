---
name: document-findings
description: Write GitHub research findings into a planning/ folder in the current repository, always date-stamped so results stay interpretable as the tooling landscape evolves. Creates planning/ if it does not already exist. Records the exact gh queries used so the research is reproducible.
---

# Document GitHub Research Findings

The tooling landscape on GitHub evolves quickly — a top project today may be abandoned in six months, and new entrants appear constantly. Any research write-up must be **date-stamped** so future readers (including future-you) know the context in which the recommendation was made.

## When to use

Trigger phrases include:
- "Save these findings"
- "Document this research"
- "Write this to the planning folder"
- "Add a note in the repo about this research"

## Procedure

### 1. Locate or create the planning folder

In the current repository (working directory), look for one of:
- `planning/`
- `docs/planning/`
- `.planning/`

If none exist, create `planning/` at the repo root.

Inside the planning folder, create or reuse a subfolder:

```
planning/github-research/
```

This keeps research artefacts grouped and out of the way of general planning docs.

### 2. Name the file with the date

File naming convention:

```
planning/github-research/YYYY-MM-DD-<short-topic-slug>.md
```

Examples:
- `planning/github-research/2026-04-16-mcp-server-frameworks.md`
- `planning/github-research/2026-04-16-markdown-to-pdf-cli.md`

Use `date +%Y-%m-%d` to get today's date — never hard-code a year.

If a file for the same topic already exists on the same date, append a short suffix (e.g. `-v2`) rather than overwriting.

### 3. Write the document

Use this template:

```markdown
# GitHub Research: <Topic>

**Research date:** YYYY-MM-DD
**Researcher:** Claude Code (github-research plugin)
**Goal:** <one-sentence statement of what the user was trying to find / decide>

> ⚠️ Note on freshness: GitHub's tooling landscape evolves quickly. Findings below reflect repository state on the research date. Re-run the queries before acting on recommendations more than ~3 months old, especially for AI/ML projects.

## Queries run

Exact `gh` / `gh api` commands used, so the research is reproducible:

```bash
<paste each command>
```

## Top candidates

| Repo | Stars | Last push | Open issues | License | Language | Notes |
|------|------:|-----------|------------:|---------|----------|-------|
| owner/repo | 1.2k | 3 days ago | 14 | MIT | Python | Active, strong CI |
| ... | ... | ... | ... | ... | ... | ... |

## Detailed evaluation

### owner/repo

- **Verdict:** recommend / maybe / avoid
- **Strengths:** ...
- **Concerns:** ...
- **Fit for our use case:** ...

(Repeat per candidate)

## Recommendation

<Concrete, opinionated recommendation. Which to try first, or "none are a good fit — build from scratch, here's why".>

## Open questions / follow-ups

- <things the user should check before committing>
```

### 4. Commit hygiene (do not auto-commit)

Write the file only. Do **not** run `git add` / `git commit` unless the user explicitly asked. Let the user review the document first.

After writing, report the path of the created file so the user can open it.

### 5. Link back from the repo's CLAUDE.md (optional)

If the repo has a `CLAUDE.md` and the user indicates this research will inform ongoing work, offer (don't auto-apply) to add a line under a "Prior research" section:

```markdown
## Prior research
- [YYYY-MM-DD: <Topic>](planning/github-research/YYYY-MM-DD-<slug>.md)
```

## Guidance

- **Always include the date** in both the filename and the document body. The whole point of this skill is reproducibility across time.
- **Always include the exact queries run.** Without them the findings cannot be re-verified when they age.
- **Be honest about limits.** If the search was narrow, say so. If results were sparse, say so. Do not invent projects to pad the list.
- **Prefer opinionated recommendations** over neutral catalogues. The user asked for research to make a decision, not a Wikipedia article.
