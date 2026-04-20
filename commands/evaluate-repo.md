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
   gh api repos/<owner>/<repo>/issues -f state=open -f per_page=30 -f sort=created -f direction=asc
   gh api repos/<owner>/<repo>/issues -f state=closed -f per_page=20
   gh api repos/<owner>/<repo>/contributors -f per_page=10
   gh api repos/<owner>/<repo>/readme --jq '.content' | base64 -d
   ```

   Also check for: `LICENSE`, a `CHANGELOG.md`, CI status badges in the README.

   **Issues-tab audit (MANDATORY — do not skip).** The Issues tab is the single best signal of whether a repo is quietly rotting. Always perform this check and surface findings in the report:

   - Count open vs. closed issues and compute the ratio.
   - Look at the **oldest open issues** (sorted ascending by creation date) — how old is the oldest unresolved bug? Anything open for 1+ years with no maintainer reply is a red flag.
   - Filter for bug-like labels (`bug`, `defect`, `crash`, `regression`) and scan titles for recurring themes — same subsystem breaking repeatedly, the same symptom reported by multiple users over years, etc.
   - Check whether maintainers **respond** to bug reports (even a triage comment counts) vs. letting them sit untouched. A graveyard of zero-comment bug reports is worse than a high issue count with active discussion.
   - Look for issues closed as `not planned` / `stale` / auto-closed by bots without resolution — this is a pattern where bugs are "dealt with" administratively rather than fixed.
   - Call out explicitly if there is **a pattern of bugs that are never dealt with**. This is a primary signal to steer the user away from the repo, even if other axes look healthy.

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
   - **Issues-tab health** — findings from the mandatory audit above. Flag as ❌ if there's a clear pattern of unresolved bugs, unanswered bug reports, or bugs closed without fixes.
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
