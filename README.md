# Github Explorer Plugin

Semantic GitHub repo discovery for finding reusable open-source components. The plugin runs `gh api search/repositories` and has Claude parse the results — ranking by star count, recent activity, maintenance signals, license, and language fit.

"Semantic" here = Claude reads and interprets the results (READMEs, metadata, stats). No vector index.

## Use case

You're working on a project and want to ask: *"Is there an open-source repo we could use for this?"* — this plugin answers that, with ranked shortlists, quick overviews, and deeper evaluations.

## Commands

- `/github-explorer:find-component` — keyword search, returns a ranked shortlist.
- `/github-explorer:overview-repo` — quick overview of a single repo (what it is, who maintains it, is it alive).
- `/github-explorer:evaluate-repo` — deep evaluation of one repo against your project's needs.
- `/github-explorer:recommend-for-project` — reads the current project's stack/needs from `CLAUDE.md` or the user's brief and returns top recommendations.
- `/github-explorer:create-resources-index` — bundle findings (from this session or a supplied list) into a curated resources-index repo. Categorised, sorted, and presented per a user-defined spec (show stars or not, ordering, badges, public/private).
- `/github-explorer:ingest-to-index` — append new findings to an existing resources index. Dedupes, enriches, classifies per the index's existing scheme, and updates `README.md` + `repos.json` + any filtered views.
- `/github-explorer:explore-to-public-index` — one-shot: search a topic, rank, confirm, and bundle the shortlist into a **public** resources-index repo (find-component + create-resources-index, visibility locked to public).
- `/github-explorer:explore-to-private-index` — same bundled workflow, scaffolding a **private** resources-index repo for personal reference.

## Agents

- `github-explorer:component-scout` — sub-agent for multi-repo evaluation passes (shortlist → evaluate each → aggregate).

## Ranking signals

Applied post-fetch by Claude:
- **Popularity** — star count, fork count
- **Activity** — last commit date, recent release cadence, open vs closed issue ratio
- **Maintenance** — stale PRs, issue response times, number of maintainers
- **Fit** — language/stack match, license compatibility, described scope vs user need
- **Red flags** — archived, one-commit repos, dead README, unclear license

## Installation

```bash
claude plugins marketplace add danielrosehill/Claude-Code-Plugins
claude plugins install github-explorer@danielrosehill
```

## Requirements

- `gh` CLI installed and authenticated
- Authenticated requests get 5,000/hour rate limit

## License

MIT
