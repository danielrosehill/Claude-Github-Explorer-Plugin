---
description: Bundle GitHub findings from this session (or a supplied list of repos) into a curated resources index repository — categorised, sorted, and presented according to a user-defined spec. Public or private.
allowed-tools: Bash, Read, Write, Edit, WebFetch
---

# Create Resources Index

Turn a set of GitHub findings into a standalone resources-index repo. Parallels the private `new-index-repo` skill (which scaffolds indices of Daniel's own repos), but operates on arbitrary third-party GitHub findings — the kind surfaced by `/github-explorer:find-component`, `/github-explorer:recommend-for-project`, or `/github-explorer:evaluate-repo`.

Reference design: a curated, topic-scoped index like `OSINT-Tools-Private` — a single canonical metadata file, a human-facing `README.md`, and optional filtered views under `lists/`.

## When to use

- User says "bundle these into an index", "make a resources repo", "collect these findings into a curated list", "spin up a {topic} awesome list".
- User has just run one or more explorer commands and wants to crystallise the findings into a shareable or archival artefact.
- User provides a flat list of GitHub URLs and wants them organised into a presented index.

Do **not** use this for indices of Daniel's own repos — route those to `new-index-repo` in the private `private-repo-mgmt` plugin.

## Inputs to gather

Ask for anything not already clear from the conversation. Batch questions — one round, not a drip.

1. **Topic / title** — what the index covers (e.g. "AI coding agents", "Self-hosted analytics").
2. **Repo name** — Train-Case-With-Hyphens (e.g. `AI-Coding-Agents-Index`). Default: `{Topic}-Index`.
3. **Source of entries** — one of:
   - Findings from the current session (list them back for confirmation)
   - A pasted list of `owner/repo` slugs or GitHub URLs
   - A file path containing one slug/URL per line
   - **GitHub topic URLs** (e.g. `https://github.com/topics/llm-agents`) — if supplied, capture them as **Sources Used** and render them in the README (see step 4). Three or fewer: render near the top under the intro. Four or more: render at the bottom in a `## Sources Used` section, so they don't obscure the recommendations.
4. **Visibility** — public or private. **Always ask explicitly and echo the choice back before creating the remote.** Never assume. Once created, state the visibility clearly in the final report.
5. **Location** — default `~/repos/github/my-repos/<Repo-Name>/`.
6. **Presentation spec** (the defining input — do not skip):
   - **Stargazer count**: show ⭐ count inline? (yes/no, default yes)
   - **Last-pushed date**: show? (yes/no, default yes)
   - **License badge**: show? (yes/no, default yes if heterogeneous licenses)
   - **Language tag**: show? (yes/no, default yes)
   - **Description source**: use upstream GitHub description, or let Claude rewrite for disambiguation? (default: rewrite when two entries would read as near-duplicates)
   - **Categorisation scheme**: one of
     - By category / tool family (default)
     - By task / use-case
     - By capability tier (e.g. beginner / advanced / power)
     - Flat alphabetical (no sections)
     - Custom — user supplies sections
   - **Section ordering**: alphabetical (default, strongly preferred), by size, manual
   - **Ordering within sections**: alphabetical (default, strongly preferred), by stars, by recency, manual

Default presentation (Daniel's standing preference — apply unless the user overrides): alphabetical sections, alphabetical within sections, `flat` badges, stars + last-pushed + license + language shown, `repos.json` included.
   - **Badges style**: `flat` (default) or `for-the-badge`
   - **Include `repos.json`?** — a machine-readable metadata file alongside the README (default: yes; enables future `/ingest-osint-links`-style workflows). Skip only if user wants a minimal README-only repo.
   - **Include filtered views under `lists/`?** — per-category / per-task markdown files generated from `repos.json` (default: no; offer if the index will grow past ~30 entries).

Echo the spec back as a compact summary before scaffolding so the user can course-correct in one turn.

## Procedure

### 1. Enrich metadata for every entry

For each `owner/repo` slug:

```bash
gh api repos/<owner>/<repo> --jq '{full_name, html_url, description, stargazers_count, forks_count, open_issues_count, pushed_at, updated_at, archived, language, license: .license.spdx_id, topics, homepage}'
```

Batch these in parallel where practical. Cache results — you will reference them multiple times.

Flag archived or inactive (`pushed_at` > 18 months) repos to the user before including them. Offer to drop or annotate.

### 2. Classify entries per the spec

- If the scheme is **by category / task / tier**, assign each entry to exactly one primary bucket plus zero-or-more secondary tags. Propose the bucket set from the entries themselves; show it to the user for approval before writing.
- If the scheme is **flat alphabetical**, skip bucketing.
- If the scheme is **custom**, use the user-supplied sections and ask for fallback handling of entries that don't fit.

Descriptions: if two entries in the same bucket would read as near-duplicates from their upstream descriptions, rewrite each to disambiguate (what it's for, who it's for, what it does differently). Do not paste ambiguous upstream blurbs verbatim.

### 3. Scaffold the repo

Directory layout:

```
<Repo-Name>/
├── README.md
├── .gitignore
├── repos.json                # if spec.include_json
├── LICENSE                   # CC BY 4.0 for public, omit for private
└── lists/                    # if spec.include_filtered_views
    ├── by-category/
    └── by-task/
```

**`.gitignore`**: `/private/`, `.env`, `.env.*`, `.DS_Store`, `node_modules/` (if any tooling is added later).

### 4. Write `README.md`

Template (adapt sections per spec):

```markdown
# {Index Title}

{1–2 sentence description of what this index covers and who it's for.}

**Last Updated:** {Month YYYY}
**Entries:** {N}

{If 1–3 GitHub topic URLs were supplied as sources, render here as:
**Sources Used:** [topic-slug](url) · [topic-slug](url)
If 4+ topic URLs, omit here and render the `## Sources Used` section at the bottom of the README instead.}

---

## Table of Contents

- [{Section}](#section-slug)
- ...

---

## {Section Name}

### {Display Name}

{One-line description — rewritten if needed for disambiguation.}

{badges line — per spec}

[{owner/repo}](https://github.com/{owner}/{repo})

### {Next Entry}
...

---

{If 4+ GitHub topic URLs were supplied as sources, render the section below; otherwise omit.}

## Sources Used

This index was assembled by surveying the following GitHub topic pages:

- [{topic-slug}](https://github.com/topics/{topic-slug})
- ...

---

## License

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
```

**Badge line construction** — emit only what the spec requests, space-separated:

- Stars: `![Stars](https://img.shields.io/github/stars/{owner}/{repo}?style={style})`
- Last commit: `![Last commit](https://img.shields.io/github/last-commit/{owner}/{repo}?style={style})`
- License: `![License](https://img.shields.io/github/license/{owner}/{repo}?style={style})`
- Language: `![Language](https://img.shields.io/github/languages/top/{owner}/{repo}?style={style})`

Hard rules:

- **Display names normalised** — human-readable project name with spaces in the `###` heading; slug only inside the URL.
- **Descriptions disambiguate** — see step 2.
- **Ordering** — respect the spec; default alphabetical by display name within each section, sections alphabetical.
- **No vanity badges** beyond what the spec requested.
- **Archived entries** — if kept, annotate inline: `> **Archived** — last push {date}. Kept for reference.`

### 5. Write `repos.json` (if spec.include_json)

Follow the `OSINT-Tools-Private` shape: a JSON object keyed by `owner/repo` slug, each value carrying the enriched metadata from step 1 plus classification fields:

```json
{
  "owner/repo": {
    "full_name": "owner/repo",
    "html_url": "...",
    "description": "...",
    "stargazers_count": 1234,
    "pushed_at": "2026-03-01T12:34:56Z",
    "archived": false,
    "language": "Python",
    "license": "MIT",
    "topics": ["..."],
    "categories": ["..."],
    "tasks": ["..."],
    "tier": "...",
    "added_at": "2026-04-22"
  }
}
```

### 6. Write filtered views (if spec.include_filtered_views)

Generate one markdown file per category and per task under `lists/by-category/` and `lists/by-task/`. Each file is a flat alphabetical list of entries tagged with that category/task. Note in the README that these are generated views.

### 7. Initialise git and create the remote

```bash
cd <path>
git init
git add .
git commit -m "Initial scaffold: {Topic} resources index ({N} entries)"
gh repo create danielrosehill/<Repo-Name> --{public|private} --source=. --remote=origin --push
```

For **private** repos, skip the `LICENSE` file and the CC BY 4.0 badge.

### 8. Report

Print:
- Local path
- GitHub URL
- Entry count, section count
- Any entries flagged as archived / inactive that were kept or dropped
- Suggested next steps (register in a master index, add more entries via follow-up runs)

## Notes

- This command is **additive and append-friendly** — a second run against an existing index should detect it, read `repos.json` (if present), dedupe, and append new entries rather than overwriting. Ask the user before rewriting an existing README from scratch.
- If the user has a personal master-index system (Daniel does — see `Private-GH-Index-Maintainer`), **do not auto-register** this new index there. Public third-party resource indices don't belong in the personal-repos master. Mention the option and let the user decide.
- Keep the README the source of truth for humans, `repos.json` the source of truth for automation. Never let them drift — if you edit one, regenerate the other.
- Respect the spec literally. If the user said "no stars", do not sneak a star badge in because it "looks nicer".
