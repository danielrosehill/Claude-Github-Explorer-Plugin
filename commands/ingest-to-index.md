---
description: Ingest one or more GitHub repos into an existing resources index created by /github-explorer:create-resources-index. Dedupes against repos.json, enriches metadata, classifies per the index's scheme, and updates README.md + filtered views.
allowed-tools: Bash, Read, Write, Edit, WebFetch
---

# Ingest to Index

Append new GitHub findings to an existing resources-index repo (the kind produced by `/github-explorer:create-resources-index`, or the reference `OSINT-Tools-Private` layout).

## When to use

- User says "add these to the index", "ingest these repos", "append to the {topic} index", "update the index with these findings".
- User provides a list of `owner/repo` slugs or GitHub URLs and names (or implies) an existing index repo.
- Triggered after a `/find-component` or `/recommend-for-project` run when the user wants to crystallise new finds into an existing index rather than a fresh one.

Do **not** use this to create a new index — route to `/github-explorer:create-resources-index` for that.

## Inputs to gather

1. **Target index** — which index repo to append to. Accept a local path, a GitHub URL, or a short name if unambiguous. If ambiguous, list candidates under `~/repos/github/my-repos/` that look like indices (have `repos.json` or a README with a clear "index" shape) and ask.
2. **New entries** — list of `owner/repo` slugs or GitHub URLs. Accept:
   - Pasted list
   - File path with one per line
   - Implicit reference to "the findings above" from the current session
3. **Classification hints** — optional. User may assign sections up-front; otherwise classify per the existing index's scheme.

## Procedure

### 1. Locate and read the target index

- Resolve the local path. `cd` there.
- Read `.claude-plugin/plugin.json` is **not** expected — this is a resources index, not a plugin.
- Read the existing `README.md`, `repos.json` (if present), and any files under `lists/` (if present).
- Infer the index's presentation spec from what's there:
  - Badge set in use (stars / last-commit / license / language / none)
  - Badge style (`flat` vs `for-the-badge`)
  - Section scheme (by category / by task / by tier / flat alphabetical / custom)
  - Ordering (alphabetical sections + alphabetical within — default; respect whatever the existing file demonstrates)
  - Whether filtered `lists/` views exist
- Confirm visibility by checking the GitHub remote: `gh repo view --json visibility`. State it in the ingest summary.

If the index lacks `repos.json`, offer to generate one from the existing README as a one-time backfill before ingesting. Makes future ingestion reliable.

### 2. Dedupe and enrich

For each new slug:

- Normalise to `owner/repo` form (lowercase-preserving — keep GitHub's case).
- Check `repos.json` — skip if already present. Report skips at the end, don't silently drop.
- Fetch metadata:
  ```bash
  gh api repos/<owner>/<repo> --jq '{full_name, html_url, description, stargazers_count, forks_count, open_issues_count, pushed_at, updated_at, archived, language, license: .license.spdx_id, topics, homepage}'
  ```
- Batch in parallel where practical.
- Flag archived or stale (`pushed_at` > 18 months ago) entries to the user before inclusion. Offer to skip, include with an "Archived" annotation, or drop.

### 3. Classify

For each new entry, assign section(s) per the index's existing scheme. Prefer reusing existing section names — do not invent new sections unless the entry genuinely doesn't fit any current bucket. If several new entries share a coherent new theme, propose a new section to the user before writing.

Rewrite descriptions to disambiguate if two entries in the same section would read as near-duplicates (same rule as `create-resources-index`).

### 4. Update `repos.json`

Append new entries with the enriched metadata plus `added_at: <today ISO date>` and the assigned classification fields. Preserve existing entries verbatim. Sort keys consistently with how the file is currently sorted (alphabetical by slug is the default).

### 5. Update `README.md`

- Insert each new entry under its assigned section, maintaining the existing ordering convention (default: alphabetical within sections, alphabetical sections).
- If a new section was created, insert it at its alphabetical position and add a Table of Contents entry.
- Update the `Last Updated:` date to the current month/year.
- Update the `Entries:` count if that field is present.
- **Do not** reorder or rewrite existing entries. Ingest is additive.

Follow the existing badge set and style exactly — do not add badges the index doesn't already use.

### 6. Update filtered views (if `lists/` exists)

Regenerate the affected per-category / per-task files from `repos.json`. Do not touch files for categories that had no new entries.

### 7. Optional: append an audit entry

If the README has a "Batch" / "Changelog" / "Ingestion log" section (as `OSINT-Tools-Private` does), append a new batch entry above the existing ones with today's date and the list of added entries. If no such section exists, skip — don't invent one.

### 8. Commit and push

```bash
git add -A
git diff --cached --stat   # show the user what's about to go out
git commit -m "Ingest N entries: <short topic summary>"
git push
```

Stage explicitly — do not blind-`git add -A` without showing the diff stat first in case unrelated files changed.

### 9. Report

- Visibility of the index (public/private) and remote URL
- Entries added (with section assignments)
- Entries skipped as duplicates
- Entries flagged archived/stale and how they were handled
- Any new section created
- Commit hash and push status

## Notes

- Ingest is **additive and idempotent** — running twice with the same list should be a no-op after the first run.
- If `repos.json` and the README disagree on what's in the index, stop and ask before proceeding. Silent reconciliation loses data.
- Never reorder existing entries to "tidy up" during an ingest — that conflates two operations and makes diffs unreviewable. If the index's ordering has drifted, offer a separate cleanup pass.
- Respect the index's existing presentation spec literally, even if it differs from your defaults.
