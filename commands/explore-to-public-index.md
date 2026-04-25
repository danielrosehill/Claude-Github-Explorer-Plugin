---
description: One-shot workflow — search GitHub for a topic, rank the findings, then bundle them into a **public** resources-index repo. Combines `find-component` and `create-resources-index` with visibility pre-set to public.
allowed-tools: Bash, Read, Write, Edit, WebFetch
---

# Explore To Public Index

Bundled workflow: take a topic or search brief from the user, discover candidate GitHub repos, and crystallise the shortlist into a **public** resources-index repo on the user's GitHub account.

This is `/github-explorer:find-component` → `/github-explorer:create-resources-index` fused into a single turn, with visibility locked to `public`. The output is intended for sharing — an "awesome-list"-style curated index.

## When to use

- User says "build a public index of X", "make me an awesome list for X", "find the best Y and publish them as an index".
- User wants a shareable, archival artefact rather than a one-off shortlist in chat.
- Route to `explore-to-private-index` instead if the index is for the user's private reference only.

## Procedure

### 1. Gather inputs in one round

Ask only what is not already clear. Batch questions — one exchange, not a drip.

- **Topic / search brief** — what the index covers (keywords, problem description, or task).
- **Repo name** — Train-Case-With-Hyphens recommended. Default: `{Topic}-Index`.
- **GitHub owner** — the account or org under which the repo will be created. Default: the authenticated `gh` user.
- **Constraints** — language, license, activity floor, shortlist size (default 15–25).
- **Presentation spec** — confirm or override the defaults:
  - Alphabetical sections, alphabetical within sections
  - Badges: stars + last-pushed + license + language, `flat` style
  - `repos.json` included
  - Categorisation: by category / tool family (propose buckets from the findings)
  - Rewrite descriptions only where two entries would otherwise read as near-duplicates
- **Location** — default `./<Repo-Name>/` under the current working directory, or a path the user supplies.

Echo the spec back as a compact summary before proceeding.

### 2. Explore

Run the `find-component` procedure with a shortlist size large enough to give the index body (default 15–25):

```bash
gh api -X GET search/repositories \
  -f q="<expanded query> stars:>10 pushed:>2024-01-01" \
  -f sort=stars -f order=desc -f per_page=50
```

Tune filters to the brief (language, license, star floor). Broaden once if the first query returns too few usable hits.

For each candidate, collect the full metadata set (`full_name`, `html_url`, `description`, `stargazers_count`, `forks_count`, `open_issues_count`, `pushed_at`, `archived`, `language`, `license.spdx_id`, `topics`).

Rank by fit, maintenance, and sensibility — drop archived / single-commit / off-topic entries unless the user asked to keep them.

### 3. Present the shortlist for confirmation

Show the ranked list — `owner/repo` · ⭐ · last push · license · language · one-line summary. Ask:

- Any entries to drop?
- Any missing repos the user knows about and wants added?
- Confirm bucket proposal (if categorisation is by category/task/tier).

Do not proceed to scaffolding until the user confirms. A public index is visible to the world — one confirmation round is cheap insurance.

### 4. Bundle into a public resources-index repo

Invoke the `create-resources-index` procedure with:

- `visibility = public` (locked — do not ask again)
- the confirmed entry list
- the agreed presentation spec
- `include_json = true` unless the user explicitly opted out

Follow that command's scaffolding rules exactly:

- Directory layout with `README.md`, `.gitignore`, `repos.json`, `LICENSE` (CC BY 4.0 recommended for public curated lists; respect an explicit override from the user).
- README with normalised display names, disambiguated descriptions, badge lines matching the spec, sections and entries alphabetised.
- `repos.json` as the machine-readable source of truth, keyed by `owner/repo`.
- `git init`, initial commit, then:

  ```bash
  gh repo create <owner>/<Repo-Name> --public --source=. --remote=origin --push
  ```

### 5. Report

Print:

- Local path
- GitHub URL (public)
- Entry count, section count
- Any archived / inactive entries kept or dropped
- Suggested next steps — add more entries via a follow-up `create-resources-index` run, or register the index elsewhere if the user maintains a master list

## Notes

- Visibility is **public** — state this explicitly in the final report. If the user changes their mind mid-run and asks for private, switch to `explore-to-private-index` rather than silently flipping the flag.
- Respect the presentation spec literally — see the hard rules in `create-resources-index`.
- If a second run targets an existing index repo, detect it, read `repos.json`, dedupe, and append rather than overwrite.
