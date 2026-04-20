---
description: Quick overview of a single GitHub repo — what it is, who maintains it, is it alive, should you care. Short and punchy, not a deep eval.
allowed-tools: Bash, Read, WebFetch
---

# Overview Repo

Give the user a fast, high-signal snapshot of one GitHub repo.

## What to do

1. **Accept the input.** The user will provide a `owner/repo` or a full GitHub URL. Normalise to `owner/repo`.

2. **Fetch core metadata:**

   ```bash
   gh api repos/<owner>/<repo>
   gh api repos/<owner>/<repo>/commits -f per_page=1
   gh api repos/<owner>/<repo>/releases -f per_page=3
   ```

   Optionally fetch the README:

   ```bash
   gh api repos/<owner>/<repo>/readme --jq '.content' | base64 -d | head -200
   ```

3. **Produce a compact overview** with these sections (2–4 lines each):
   - **What it is** — one-sentence description grounded in the README, not the API description alone.
   - **Who maintains it** — owner type (user/org), number of contributors if readily visible, notable maintainers.
   - **Signals of life** — last commit, last release, open-issue ratio, archived/public/private.
   - **Install / usage** — the one install command or import snippet if the README surfaces one.
   - **Verdict** — one line: actively maintained / dormant / archived / toy project / production-grade.

4. **Be honest.** If the README is padding and the repo has two commits, say so. If it's a fork of a popular project, say so.

## Notes

- Don't dump the full README — the user wants a snapshot, not a reprint.
- Don't compute a "fit for my project" score here — that's what `/evaluate-repo` is for.
- If the repo doesn't exist or is private, say so and stop.
