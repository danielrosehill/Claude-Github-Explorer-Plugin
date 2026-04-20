---
description: Recommend open-source GitHub repos that fit the current project. Reads the repo's CLAUDE.md / manifest files to infer stack and needs, then surfaces top candidates with a rationale per pick.
allowed-tools: Bash, Read, WebFetch, Grep, Glob
---

# Recommend for Project

Suggest open-source repos that would slot into the user's current project. Contextual — driven by what this repo already looks like.

## What to do

1. **Understand the project.**
   - Read `CLAUDE.md`, `README.md` at the repo root.
   - Read the primary manifest (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.) — note language, existing deps, framework choices.
   - Glance at `src/` or the main entry point for the overall shape.

2. **Anchor on a need.** The user will usually say what they're looking for ("need something for PDF generation"). If they didn't, ask — don't guess across an entire project.

3. **Search with stack-aware filters.** Use `gh api search/repositories` with:
   - `language:` matching the project's primary language
   - `license:` filtered to a compatible set when the project has a declared license
   - Keyword expansion appropriate to the need

4. **Rank top candidates** (5–7 max). Apply the same signals as `/find-component` (popularity, activity, license, fit), but weight **stack fit** heavier — a brilliant Ruby library is useless to a Go project.

5. **For the top 2–3 candidates, pull a bit of extra data** (last release, open-issue count, one-line of the README) so each recommendation has enough flesh to decide on.

6. **Output shape:**

   ```markdown
   ## Project context
   <one paragraph — what you inferred about the project and the need>

   ## Recommendations

   ### 1. owner/repo — ⭐ N · last push · license
   <1–2 sentences on what it is and why it fits this project specifically>

   ### 2. ...

   ## Honourable mentions
   <1–3 bullets for things that almost made the cut>

   ## Next step
   Run `/github-explorer:evaluate-repo <owner/repo>` for a deeper look at any candidate.
   ```

## Notes

- Fit-to-this-project is the whole point. Don't just return "popular X libraries" — tie each pick to something concrete in the project.
- If the project has an anti-pattern constraint (e.g. "no JS build step", "pure Python stdlib only"), honour it strictly.
- If nothing good fits, say so. A real "build this yourself, the ecosystem is thin here" is more useful than a weak recommendation.
