---
description: Deep evaluation of a single GitHub repo as a candidate for the user's project — fit, maintenance, risk, integration effort. Longer and more opinionated than overview-repo.
allowed-tools: Bash, Read, WebFetch, Grep, Glob
---

# Evaluate Repo

Deeply evaluate one candidate repo against the user's project needs. Output a structured verdict the user can act on.

## What to do

1. **Gather context.**
   - Target repo: `owner/repo` from the user.
   - The user's project: read the current repo's `CLAUDE.md`, `README.md`, `package.json`/`pyproject.toml`/`Cargo.toml` (whatever exists) to infer stack, existing deps, and constraints.
   - If the user described a specific need in their prompt, let that drive the fit question — don't second-guess.

2. **Fetch detailed repo data:**

   ```bash
   gh api repos/<owner>/<repo>
   gh api repos/<owner>/<repo>/commits -f per_page=10
   gh api repos/<owner>/<repo>/releases -f per_page=5
   gh api repos/<owner>/<repo>/issues -f state=open -f per_page=20
   gh api repos/<owner>/<repo>/contributors -f per_page=10
   gh api repos/<owner>/<repo>/readme --jq '.content' | base64 -d
   ```

   Also check for: `LICENSE`, a `CHANGELOG.md`, CI status badges in the README.

3. **Evaluate against these axes** (mark each ✅ / ⚠️ / ❌ with a one-line justification):

   **Fit**
   - Solves the user's described problem?
   - Language/runtime match with the user's project?
   - API surface reasonable for what the user needs?
   - Footprint sensible (not dragging in 500 MB of deps for a 10-line utility)?

   **Maintenance**
   - Last commit recency
   - Release cadence
   - Open-issue ratio and response times (skim a few recent issues)
   - Bus factor — is it one person?

   **Risk**
   - License compatibility with the user's project
   - Known security/CVE history visible in issues
   - Breaking-change frequency
   - Archived or deprecation notices

   **Integration effort**
   - Install/setup complexity
   - Quality of documentation
   - TypeScript/typing availability where relevant
   - Test suite presence

4. **Produce the evaluation report** as markdown:

   ```markdown
   # Evaluation: owner/repo

   **TL;DR:** <one-line recommendation — adopt / try / avoid>

   ## Fit
   - ✅/⚠️/❌ <axis> — <justification>

   ## Maintenance
   - ...

   ## Risk
   - ...

   ## Integration effort
   - ...

   ## Alternatives worth considering
   <1–3 adjacent repos if you spot obvious ones, else "none obvious">

   ## Recommendation
   <2–3 sentences>
   ```

5. **Don't hedge everything.** If it's the right tool, say so. If it's abandoned and you'd regret adopting it in six months, say so.

## Notes

- If the user hasn't pointed at a specific need, ask one question to anchor the eval — otherwise the fit axis is meaningless.
- Don't invent issues or activity you didn't see. If the API didn't return something, say "not checked" rather than guessing.
