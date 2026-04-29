---
description: Given a seed GitHub repo and a description of what the user wants more of, find similar repos. The user names the angle of similarity (purpose, tech stack, UX, niche) so results aren't just "same topic tags".
allowed-tools: Bash, Read, WebFetch
---

# Find Similar Repos

The user has a GitHub repo they like and wants to find more like it. "Similar" is ambiguous on its own — pin down the angle, then search.

## What to do

1. **Get the seed.** The user should have given a repo (URL or `owner/repo`) and a short description of what they want more of. If the angle is missing, ask one question: *"What aspect should the matches share — same purpose, same tech stack, same UX/feel, same niche, or something else?"* Don't ask more than that.

2. **Profile the seed repo** via `gh`:

   ```bash
   gh api repos/<owner>/<repo>
   gh api repos/<owner>/<repo>/topics
   gh api repos/<owner>/<repo>/readme --jq '.content' | base64 -d | head -c 4000
   ```

   Capture: `description`, `topics`, `language`, `license`, primary purpose (from README), notable framework/runtime hints.

3. **Build the search query from the angle the user gave**:
   - **"Same purpose"** — search the description's key nouns + a couple of seed topics. Don't over-constrain by language.
   - **"Same tech stack"** — anchor on `language:` and the framework topics (e.g. `topic:nextjs topic:tailwindcss`). Loosen on purpose.
   - **"Same UX/feel"** — search the visible product category words (e.g. "terminal dashboard", "minimal markdown editor"); language matters less.
   - **"Same niche"** — lean on the most specific topic tag(s) and README jargon.
   - Always exclude the seed itself from the results.

   Example call:

   ```bash
   gh api -X GET search/repositories \
     -f q="<angle-tuned query> stars:>10 pushed:>2024-06-01 -repo:<owner>/<repo>" \
     -f sort=stars -f order=desc -f per_page=30
   ```

   If the seed has very specific topics, also try a topic-only pass:

   ```bash
   gh api -X GET search/repositories \
     -f q="topic:<t1> topic:<t2> stars:>10 -repo:<owner>/<repo>" \
     -f sort=stars -f order=desc -f per_page=30
   ```

   Merge and dedupe the two passes.

4. **Rank by similarity to the seed along the chosen angle**, not just stars. For each candidate collect `full_name`, `description`, `stargazers_count`, `pushed_at`, `archived`, `language`, `topics`, `license.spdx_id`. Score with judgement:
   - Strong overlap with the seed on the requested angle (heaviest signal)
   - Active maintenance (pushed within ~6 months)
   - Reasonable stars for the niche
   - Penalise: archived, single-commit, off-topic, obvious forks of the seed (unless the user wants forks)

5. **Produce a shortlist** of 5–10 repos. For each:
   - `owner/repo` — ⭐ stars · last push · license · language
   - One-line summary
   - One-line "how it's similar to <seed>" — tied to the angle the user named
   - Verdict: "closest match" / "strong fit" / "adjacent" / "tangential, FYI"

6. **End with a recommendation.** Name the top 1–2 closest matches and why. Offer `/github-explorer:evaluate-repo <url>` for a deeper look or `/github-explorer:component-scout` for a side-by-side comparison.

## Notes

- The angle the user named is the spine of this skill — don't quietly drift back to "same topic tags" if they asked for "same UX".
- If the seed is itself very generic (e.g. `awesome-*` lists, mega-frameworks), say so and ask the user to narrow before searching — otherwise results will be noise.
- Never return forks of the seed as "similar" unless the user explicitly wants them.
- One or two search calls is plenty; don't burn rate limit on permutations.
