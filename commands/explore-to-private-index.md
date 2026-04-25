---
description: One-shot workflow — search GitHub for a topic, rank the findings, then bundle them into a **private** resources-index repo. Combines `find-component` and `create-resources-index` with visibility pre-set to private.
allowed-tools: Bash, Read, Write, Edit, WebFetch
---

# Explore To Private Index

Bundled workflow: take a topic or search brief from the user, discover candidate GitHub repos, and crystallise the shortlist into a **private** resources-index repo on the user's GitHub account.

This is `/github-explorer:find-component` → `/github-explorer:create-resources-index` fused into a single turn, with visibility locked to `private`. The output is intended for the user's personal reference — an internal knowledge base of interesting repos, not a shareable awesome-list.

## When to use

- User says "build a private index of X", "collect these findings for my own reference", "private resources bundle for Y".
- User wants an archival / lookup artefact that stays on their own account.
- Route to `explore-to-public-index` instead if the index is meant to be shared publicly.

## Procedure

### 1. Gather inputs in one round

Ask only what is not already clear. Batch questions — one exchange, not a drip.

- **Topic / search brief** — what the index covers (keywords, problem description, or task).
- **Repo name** — Train-Case-With-Hyphens recommended. Default: `{Topic}-Index-Private` (suffix makes private intent visible in listings).
- **GitHub owner** — the account or org under which the repo will be created. Default: the authenticated `gh` user.
- **Constraints** — language, license, activity floor, shortlist size (default 15–25). For a private reference index the bar can be looser — niche or experimental repos are often worth keeping.
- **Presentation spec** — confirm or override the defaults:
  - Alphabetical sections, alphabetical within sections
  - Badges: stars + last-pushed + license + language, `flat` style
  - `repos.json` included (especially important for a private index — enables future automation on top of the list)
  - Categorisation: by category / tool family (propose buckets from the findings)
  - Rewrite descriptions only where two entries would otherwise read as near-duplicates
- **Location** — default `./<Repo-Name>/` under the current working directory, or a path the user supplies.

Echo the spec back as a compact summary before proceeding.

### 2. Explore

Run the `find-component` procedure with a shortlist size large enough to give the index body (default 15–25):

```bash
gh api -X GET search/repositories \
  -f q="<expanded query> stars:>5 pushed:>2024-01-01" \
  -f sort=stars -f order=desc -f per_page=50
```

Tune filters to the brief (language, license, star floor). For a private index, a lower star floor is reasonable since discovery value often lives in the long tail.

For each candidate, collect the full metadata set (`full_name`, `html_url`, `description`, `stargazers_count`, `forks_count`, `open_issues_count`, `pushed_at`, `archived`, `language`, `license.spdx_id`, `topics`).

Rank by fit and maintenance. Archived entries can be retained in a private index — annotate them inline rather than dropping.

### 3. Present the shortlist for confirmation

Show the ranked list — `owner/repo` · ⭐ · last push · license · language · one-line summary. Ask:

- Any entries to drop?
- Any missing repos the user knows about and wants added?
- Confirm bucket proposal (if categorisation is by category/task/tier).

For a private index, one light confirmation pass is enough — the blast radius is low.

### 4. Bundle into a private resources-index repo

Invoke the `create-resources-index` procedure with:

- `visibility = private` (locked — do not ask again)
- the confirmed entry list
- the agreed presentation spec
- `include_json = true` unless the user explicitly opted out

Follow that command's scaffolding rules exactly, with private-repo adjustments:

- Directory layout with `README.md`, `.gitignore`, `repos.json`.
- **Omit the `LICENSE` file and the CC BY 4.0 badge** — private repos don't need an open-source licence notice.
- README with normalised display names, disambiguated descriptions, badge lines matching the spec, sections and entries alphabetised.
- `repos.json` as the machine-readable source of truth, keyed by `owner/repo`.
- Archived entries, if kept, annotated: `> **Archived** — last push {date}. Kept for reference.`
- `git init`, initial commit, then:

  ```bash
  gh repo create <owner>/<Repo-Name> --private --source=. --remote=origin --push
  ```

### 5. Report

Print:

- Local path
- GitHub URL (private — visible only to the user and collaborators)
- Entry count, section count
- Any archived / inactive entries kept or dropped
- Suggested next steps — add more entries via a follow-up `create-resources-index` run, or register the index in the user's personal master-index system if they maintain one

## Notes

- Visibility is **private** — state this explicitly in the final report. Never upgrade a private index to public without an explicit user instruction.
- Badges in private READMEs still render via shields.io (they fetch public metadata), so `owner/repo` slugs must be for public upstream repos — a private index of private repos is a different workflow.
- If a second run targets an existing index repo, detect it, read `repos.json`, dedupe, and append rather than overwrite.
- For the user's personal master-index workflows (if any), mention the option of registering this new index there and let the user decide — don't auto-register.
