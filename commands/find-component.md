---
description: Search GitHub for reusable open-source components matching a keyword or problem description. Returns a ranked shortlist (stars, activity, fit) with a one-line verdict per repo.
allowed-tools: Bash, Read, WebFetch
---

# Find Component

The user wants to find an open-source GitHub repo for a specific purpose. Search, rank, summarise.

## What to do

1. **Clarify the query if needed.** The user's request may be a keyword ("pdf parser"), a description ("need a lib that does X"), or a task ("something to replace $thing"). If it's vague, ask one clarifying question max — otherwise proceed.

2. **Build the search query.** Expand keywords sensibly — include likely technical synonyms. Respect constraints the user mentioned (language, license, "no heavy frameworks").

3. **Call the GitHub search API** via `gh`:

   ```bash
   gh api -X GET search/repositories \
     -f q="<expanded query> stars:>10 pushed:>2024-01-01" \
     -f sort=stars -f order=desc -f per_page=30
   ```

   Tune filters to the query:
   - Add `language:<lang>` when the user specified a language.
   - Add `license:<license>` when licensing is load-bearing.
   - Lower the `stars:` floor for niche topics.
   - Drop the `pushed:` filter only if the user is explicitly looking at older/stable projects.

4. **Parse and rank** the results. For each candidate collect:
   - `full_name`, `html_url`, `description`
   - `stargazers_count`, `forks_count`, `open_issues_count`
   - `pushed_at`, `updated_at`, `archived`
   - `license.spdx_id`, `language`, `topics`

   Rank with these weights (judgement, not a formula):
   - Obvious fit to the user's described need (heaviest signal)
   - Active maintenance (commit within ~6 months)
   - Reasonable star count for the niche
   - Sensible license
   - Penalise: archived, single-commit, no README, unclear scope

5. **Produce a shortlist** of 5–10 repos. For each:
   - `owner/repo` — ⭐ stars · last push · license · language
   - One-line summary of what it does
   - One-line verdict: "strong fit" / "worth a look" / "niche but active" / "archived, skip"

6. **End with a recommendation.** Name the top 1–2 and say briefly why. Offer the user `/github-explorer:evaluate-repo <url>` for a deeper look at any candidate.

## Notes

- Don't return more than ~10 results — the ranked shortlist is the product.
- If the API returns nothing useful, try a broader query once before giving up.
- Never silently drop candidates the user might want to see — say if you filtered something out and why.
- Respect rate limits: one search call per request is plenty.
