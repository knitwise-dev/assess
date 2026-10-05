# Changelog

All notable changes to the Knitwise Assess Action (`knitwise-dev/assess`). Versions follow [semantic versioning](https://semver.org); the major tag (`v0`) always points at the latest release in that series.

## 0.1.5

Clearer about what the assessment can and can't do, and tidier wording for private repositories on GitHub Free. Scored with **rubric v0.2.2**, as in 0.1.4: wording only, no score changes, so 0.1.4 and 0.1.5 scores compare directly.

- **Shorter paid-plan fix title:** "Enforce PRs and passing checks (needs a paid GitHub plan)"; the plan details (Pro for a personal account, Team for an organization) stay in its "Why".
- **Safety gates key evidence** on a private repository whose plan doesn't enforce branch rules now reads "N of M gates in place; branch rules not enforceable on this plan", counting the gates that could be judged. Wording only: rubric 0.2.2, no score changes.
- **Limitations:** new `docs/limitations.md` lists what the assessment can't see or measure, with workarounds. Linked from the README, the install guide and the security FAQ.
- **Install guide:** the pin placeholder reads `<commit SHA of the release>`, with the `git ls-remote` command beside it; each release's notes give the full pin line.

## 0.1.4

Correct scoring for private repositories on GitHub Free, clearer fix wording, and a pinned install. Scored with **rubric v0.2.2**: scores for private repositories on GitHub Free change from 0.1.3 (rubric v0.2.1), so don't compare them directly; see "Changes" in `docs/scoring-rubric.md`.

- **Private repositories on GitHub Free (rubric 0.2.2):** GitHub Free doesn't enforce rulesets or branch protection on private repositories. When a private repository's branch rules endpoint answers 403 asking you to upgrade, required reviews and required status checks are now not applicable instead of 0, and one fix explains that enforcing them needs GitHub Pro (personal account) or GitHub Team (organization). Any other 403 there makes both unknown, with the reason "Couldn't read branch rules (permission)". The `CODEOWNERS` evidence notes that on this plan it documents ownership but doesn't request reviews.
- **Level cap:** on such a repository the overall level is capped at 4, whether or not safety gates has a score, with a note under the level when the cap lowers it: level 5 means the practice is enforced.
- **Fix wording:** "Scan for committed secrets" now says "a secret scanner such as TruffleHog or gitleaks", and "Review new dependencies in CI" says "a dependency check such as GitHub's dependency review or OSV-Scanner", instead of naming one tool. The paid-plan fix's second line reads "Until then:".
- **Install guide:** recommends pinning the Action to a release's commit SHA (`knitwise-dev/assess@<sha> # v0.1.4`), with how to find the SHA, and `@v0` as the simpler auto-updating alternative.

## 0.1.3

Correct scoring for Java, Kotlin, Android and Python teams, and point the report at the Knitwise landing page. Scored with **rubric v0.2.1**: scores may shift from 0.1.2 (rubric v0.2) for those stacks, so don't compare them directly; see "Changes" in `docs/scoring-rubric.md`.

- **Call-to-action link:** the report's "Request early access" link now opens the Knitwise landing page's free-assessment form, https://bloreai.com/home/contact_us?topic=a+free+assessment, instead of the bloreai.com home page.
- **Test-file detection across stacks:** test discipline now recognises Android `androidTest/` folders, `cypress/`, `playwright/` and `conftest.py` as tests; counts code under `src/main/` as source even in packages named `build`, `out`, `spec` or `test`; and no longer counts Gradle `*.gradle.kts` build scripts as source. The rubric now says what counts as a test file.
- **Rubric 0.2.1:** a correction for that test-file and source-file classification. No criteria or thresholds changed, but scores may shift for Java, Kotlin, Android and Python repositories.
- **Alignment with industry frameworks:** a new rubric section shows which parts of the DORA AI Capabilities Model (2025) and which OpenSSF Scorecard checks each area relates to. It doesn't change any score.

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
