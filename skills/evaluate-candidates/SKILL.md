---
name: evaluate-candidates
description: Deep-dive evaluation of 2–5 candidate GitHub repos the user is seriously considering using, forking, or integrating. Pulls README, code-quality signals, contributor count, commit cadence, open-issue ratio, release history, and license to produce a concrete recommendation.
---

# Evaluate GitHub Candidate Repos

After a search surfaces plausible candidates, the user typically wants a close look at 2–5 finalists before committing. This skill produces a structured side-by-side evaluation using `gh api` to inspect each repo.

## When to use

Trigger phrases include:
- "Compare these repos"
- "Look closer at X and Y"
- "Which of these should I use?"
- "Evaluate this project before I fork it"

Accept candidates as either a list of `owner/repo` identifiers or the output of a previous `search-repos` / `search-ai-tools` run.

## Procedure

For each candidate, collect the following signals. Run these in parallel where possible.

### 1. Core metadata

```bash
gh api repos/<owner>/<repo> --jq '{
  full_name, description, homepage,
  stars: .stargazers_count, forks: .forks_count,
  open_issues: .open_issues_count,
  created: .created_at, pushed: .pushed_at, updated: .updated_at,
  language, license: .license.spdx_id,
  default_branch, archived, disabled,
  has_issues, has_wiki, has_discussions
}'
```

### 2. README

```bash
gh api repos/<owner>/<repo>/readme --jq '.content' | base64 -d | head -200
```

Summarize in 2–3 sentences: what it does, who it's for, and installation/usage shape (library vs CLI vs service).

### 3. Maintenance health

- **Recent commits** (last 10):
  ```bash
  gh api "repos/<owner>/<repo>/commits?per_page=10" --jq '.[] | {date: .commit.author.date, msg: .commit.message | split("\n")[0], author: .commit.author.name}'
  ```
- **Contributor count**:
  ```bash
  gh api "repos/<owner>/<repo>/contributors?per_page=100" --jq 'length'
  ```
  Solo-maintainer projects are higher risk.
- **Releases**:
  ```bash
  gh api "repos/<owner>/<repo>/releases?per_page=5" --jq '.[] | {tag: .tag_name, date: .published_at}'
  ```
  Regular releases are a strong positive signal.
- **Open vs closed issue ratio**:
  ```bash
  gh api "search/issues?q=repo:<owner>/<repo>+type:issue+state:open" --jq '.total_count'
  gh api "search/issues?q=repo:<owner>/<repo>+type:issue+state:closed" --jq '.total_count'
  ```
  A huge open pile with few closes suggests the maintainer has disengaged.

### 4. Code-quality signals

Check the tree for common markers:

```bash
gh api "repos/<owner>/<repo>/contents" --jq '.[].name'
```

Look for:
- `tests/`, `test/`, `__tests__/`, `*_test.go`, `spec/` → tests exist
- `.github/workflows/` → CI configured
- `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` → project welcomes contribution
- `CHANGELOG.md` → actively tracked history
- Lockfiles (`package-lock.json`, `poetry.lock`, `Cargo.lock`, `go.sum`) → reproducible builds
- `Dockerfile`, `docker-compose.yml` → deploy story
- Documentation site (`docs/`, mkdocs, docusaurus) → serious project

### 5. License clarity

Flag: missing license, non-OSI license, license mismatch between package metadata and repo, copyleft (GPL/AGPL) vs permissive (MIT/Apache-2.0/BSD). This matters if the user plans to integrate.

### 6. Issues: look at the last 5 open

```bash
gh api "repos/<owner>/<repo>/issues?state=open&per_page=5" --jq '.[] | {title, created: .created_at, labels: [.labels[].name], author: .user.login}'
```

Are they user-facing bugs with no maintainer response? That's a red flag.

### 7. Produce a recommendation

For each candidate output:

- **Verdict** — one of: *recommend*, *maybe*, *avoid*, with one-line reason.
- **Strengths** — 2–3 bullets.
- **Concerns** — 2–3 bullets (or fewer if clean).
- **Fit** — does it match what the user described needing?

End with a summary ranking across the candidates and a clear recommendation of which to try first, or a "none are great, here's why" honest verdict if applicable.

## Guidance

- **Don't oversell.** If none of the candidates look good, say so and suggest widening the search or building from scratch.
- **Solo-maintainer risk is real** for anything load-bearing. Note it.
- **Recent activity matters more than star count** for production decisions.
- **License problems are deal-breakers** for integration — surface these prominently, not as footnotes.
