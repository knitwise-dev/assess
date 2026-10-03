# Changelog

All notable changes to the Knitwise Assess Action (`knitwise-dev/assess`). Versions follow [semantic versioning](https://semver.org); the major tag (`v0`) always points at the latest release in that series.

## 0.1.2

Pilot feedback on 0.1.1. Still scored with **rubric v0.2**: scores are unchanged and comparable with 0.1.1.

- **Lookback-days note fixed:** the "not enough PRs" note suggests double the current `lookback-days` (at least 180), instead of always 180. When the window already reaches back before the repository's first commit, it says the window covers the full history and suggests nothing larger.
- **New unscored observation, direct pushes:** when at least 10 commits on the default branch are in the window and more than half arrived without a pull request, the report says so above Top fixes ("{direct} of {total} commits on {branch} ({pct}%) were pushed directly, without a pull request…"). `score.json` records the counts under `observations`. Only counts are read: no commit IDs, messages or authors. It never affects a score, and it's left out if the commits can't be read.
- **New optional input `max-commits`** (default 300, at most 1000): how many of the most recent default-branch commits the observation reads. Past it, the wording says "of the most recent {n} commits" and `score.json` marks the counts `sampled`. Invalid values fall back to 300 with a warning. Commits are read 25 per page, because larger pages time out on busy repositories.

## 0.1.1

Fixes from the first pilot run. Scored with **rubric v0.2**: scores from 0.1.0 (rubric v0.1) shouldn't be compared directly; see "Changes" in `docs/scoring-rubric.md`.

- **Build commands recognised in Java, Kotlin, Go, Rust and monorepos:** Maven goals (`mvn`, `mvnw`), Gradle tasks (`gradle`, `gradlew`), `go` and `cargo` commands, npm scripts in `package.json` files up to 3 folders deep (never `node_modules`), and scripts in the repository (`*.sh`, `mvnw`, `gradlew`).
- **Claude Code detected from the repository:** a committed `CLAUDE.md` or `.claude/` folder means Claude Code is in use, so the permissions and hooks criteria apply and a missing `.claude/settings.json` scores 0.
- **Single maintainer, safety gates:** with fewer than 2 human contributors, "requires reviews" is not applicable ("Single maintainer: GitHub doesn't allow approving your own PR."), and the report suggests requiring CI and a self-review checklist instead.
- **Single maintainer, review depth:** "reviewed by someone other than the author" is not applicable for a single maintainer, and the self-review checklist fix is shown once.
- **No duplicate fixes:** each fix appears once in Top fixes, and duplicates are removed before the top three are chosen.

## 0.1.0

First public release, as Knitwise Assess (`knitwise-dev/assess`), under the Apache License 2.0. Renamed product to Knitwise; licence Apache-2.0. Assesses how a team governs AI coding agents (Claude Code and Codex) from a read-only run in the team's own CI.

- **Six scored dimensions** (0–5): agent configuration, test discipline, review depth, PR hygiene, safety gates and adoption signal, plus an overall level (1–5). Every rule is in `docs/scoring-rubric.md` (rubric v0.1).
- **Report:** a Markdown report in the job summary with a dimensions table, up to three ranked fixes, and per-dimension details; written to `report.md`.
- **`score.json`:** scores, points and criterion results only, with the rubric and Action versions; no code and no names.
- **AI-assisted PR detection** (best-effort): co-author trailers, agent branch names, agent bots, labels and PR descriptions.
- **Read-only:** needs `contents: read` and `pull-requests: read`; the Action's own code calls only `api.github.com`.
- **Privacy:** developer logins are replaced with anonymous ids as they are read; nobody is named, ranked or scored.
- **Inputs:** `lookback-days`, `max-prs`, `exclude-paths`, `output-dir`, `show-cta`, and the optional `admin-token`.
- **Outputs:** `overall-level`, `report-file`, `score-file`.
