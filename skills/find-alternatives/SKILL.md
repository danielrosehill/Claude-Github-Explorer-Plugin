---
name: find-alternatives
description: Given a known GitHub project, find similar or competing projects that solve the same problem. Useful when the user has heard of one option but wants to survey the landscape before committing, or suspects a project they're using has better-maintained alternatives.
---

# Find Alternatives on GitHub

Given one reference project, produce a shortlist of projects solving the same problem — competitors, forks that took a different direction, or adjacent tools covering the same use case.

## When to use

Trigger phrases include:
- "What are the alternatives to X?"
- "Is there something similar to X but [smaller / in Go / maintained]?"
- "I'm using X but it feels abandoned — what else is out there?"
- "Show me competitors to X"

## Procedure

### 1. Inspect the reference project first

Start by pulling metadata on the reference so you know what to search for:

```bash
gh api repos/<owner>/<repo> --jq '{full_name, description, language, topics, homepage, stars: .stargazers_count, pushed_at}'
```

From this, extract:
- The **topics** (GitHub's own tags) — `.topics`
- Keywords in the description
- The primary language (if the user wants same-language alternatives)

### 2. Search using the reference's topics

Topics are the cleanest signal for "same category":

```bash
gh api -X GET search/repositories \
  -f q='topic:<topic1> topic:<topic2> -repo:<owner>/<repo> archived:false' \
  -f sort=stars -f order=desc --jq '.items[:20] | .[] | {full_name, stars: .stargazers_count, pushed_at, language, description, url: .html_url}'
```

Exclude the reference repo itself with `-repo:<owner>/<repo>`.

### 3. Keyword fallback

If topics are sparse, build a keyword query from the reference description:

```bash
gh search repos "<keywords from description>" \
  --sort stars --limit 20 \
  --json name,owner,description,stargazersCount,pushedAt,language,url
```

### 4. Find hard forks that became their own thing

Sometimes the best alternative is a fork that diverged:

```bash
gh api "repos/<owner>/<repo>/forks?sort=stargazers&per_page=20" --jq '.[] | select(.stargazers_count > 10) | {full_name, stars: .stargazers_count, pushed_at, description}'
```

A fork with meaningful stars and ongoing commits usually indicates a serious divergence worth surfacing.

### 5. Present a comparison

Include the **reference project at the top** for calibration, then list alternatives ranked by a composite of stars + recency. For each alternative:

- Full name, stars, last push
- Primary language
- One-sentence description
- A short "vs reference" note — how it differs (smaller scope, different language, more active maintenance, different license, etc.)

### 6. Offer next steps

Typical follow-ups:
- Deeper dive on 2–3 of the alternatives via `evaluate-candidates`
- Documenting the landscape via `document-findings`

## Guidance

- **Don't just list — differentiate.** The whole point of this skill is helping the user see *why* they might pick a different option.
- **Watch for forks-of-forks confusion** — sometimes the community has migrated to a descendant fork while the original repo sits abandoned.
- **Include language-native alternatives** if the user is looking to switch ecosystems (e.g. Go rewrite of a Python tool).
- Do not invent projects. Report only what the CLI returned.
