# Scoring rubric

**Version 0.2.2.** Every `score.json` records the rubric version it was scored with (`rubricVersion`), so scores from different versions are never compared by mistake.

This document explains exactly how the Knitwise by Blore.AI Assess Action turns your repository's activity into scores. It is public so you can check our working (NFR-9). The numbers below live in one file in the code, `packages/assess/src/thresholds.ts`, and change only with a new rubric version.

## How a dimension is scored

Each of the six dimensions has criteria worth points. A criterion is one of:

- **Met, partly met or not met:** it counts toward the dimension's *applicable points*.
- **Not applicable:** it does not apply to your team, for example Claude Code rules when your team uses only Codex.
- **Unknown:** the Action could not see it, for example a setting that needs admin access, which the Action never requests.

The score is **round(5 × points earned ÷ applicable points)**, rounding halves up.
*Why:* you are scored only on what applies to you and what we could see. Not-applicable and unknown criteria are listed in the report with the reason. **Unknown is never counted as 0.**

Penalties subtract points. Earned points never go below 0 or above the applicable points.
*Why:* a penalty flags a real risk without wiping out the rest of the dimension.

If a dimension has fewer than **3** applicable points, it shows **insufficient data** instead of a score.
*Why:* a score resting on one or two points says more about what we couldn't see than about your practice, and a 0 would claim something we did not measure.

## General rules

| Rule | Why |
|---|---|
| With fewer than **10** merged PRs in the lookback window, criteria based on ratios are not applicable, with the reason "Not enough PRs in window: found N, need 10." The rest of the dimension is still scored if at least 3 applicable points remain. | Percentages over a handful of PRs swing wildly and would mislead, but settings and files can still be judged. |
| Comparisons between AI-assisted and other PRs run only when each group has at least **5** PRs. | A gap between a group of 2 and a group of 50 is noise, not a finding. |
| Figures per contributor appear only when there are at least **3** contributors. Individuals are never named or scored. | With 1 or 2 people, a "share of contributors" points at a person (BR-4). |
| Only committed files count. `.claude/settings.local.json` is ignored. | Local settings are personal and not part of the team's agreed setup. |
| Wherever changed lines are counted (PR size, large PRs, silent approvals), lockfiles, generated files and test data are left out. Lockfiles: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `poetry.lock`, `Pipfile.lock`, `Cargo.lock`, `go.sum`. Generated: files your `.gitattributes` marks `linguist-generated`, plus minified files and source maps. Test data: `**/__snapshots__/**`, `**/*.snap`, `**/test/fixtures/**`, `**/tests/fixtures/**`, `**/testdata/**`, `**/test/golden/**`, `**/*.golden`. The optional `exclude-paths` input adds your own globs. | Nobody reviews a lockfile, a build bundle or a recorded fixture line by line, so counting them would make routine changes look huge. If you commit build output, mark it `linguist-generated`; GitHub then also collapses it in diffs. |

What this means for a repository with fewer than 10 merged PRs:

| Dimension | Points left | Result |
|---|---|---|
| Agent configuration | all (no ratios) | scored |
| Test discipline | 1 (CI runs tests) | insufficient data |
| Review depth | 0 | insufficient data |
| PR hygiene | 2 (template criteria) | insufficient data |
| Safety gates | all except settings the token cannot read | scored if at least 3 remain |
| Adoption signal | 0 | insufficient data |

## AI-assisted PR detection (not scored)

The Action flags a PR as AI-assisted when it finds any of these signals:

- `Co-authored-by` trailers naming Claude or Codex
- agent branch-name patterns
- bot authors
- labels
- answers in the PR template

It also reports which agents are in use: Claude Code, Codex, both, or unknown. **Claude Code is in use** if a `CLAUDE.md` (at the root or in a subfolder) or a `.claude/` folder is committed, or any PR has a Claude signal such as a co-author trailer, whatever else is detected. Codex is in use only when a PR has a Codex signal.

*Why:* several dimensions compare AI-assisted PRs with others, and the agent configuration rules depend on which agents the team uses.

Detection is best-effort: an agent used without any of these signals is invisible to it, and every report says so. The per-PR flag is used only inside the run. It never appears in the report or in `score.json`.

## Agent configuration (5 points)

| Points | Criterion | Applies when | Why |
|---|---|---|---|
| +1 | `CLAUDE.md` or `AGENTS.md` exists at the repo root with at least **10** non-empty lines | Always | Agents need a shared starting point, and a few lines is not yet a standard. |
| +1 | It names build, test or lint commands that exist in the repository: an npm script in any `package.json` up to 3 folders deep (not `node_modules`), a `Makefile` target, a `pyproject.toml` script, a Maven goal (`mvn`/`mvnw`) with a `pom.xml`, a Gradle task (`gradle`/`gradlew`) with a `build.gradle(.kts)`, `go` with a `go.mod`, `cargo` with a `Cargo.toml`, or a script in the repository (`*.sh`, `mvnw`, `gradlew`) | Always | Agents verify their work with these commands, so they must be real. With no instruction files, no commands are named. |
| +1 | Each agent in use has instructions it reads: `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex. If both files exist, they don't contradict each other on commands, or one names the other as the source of truth | Always | An agent with no instructions, or two files that disagree, gives inconsistent results. |
| +1 | `.claude/settings.json` has permission rules and no broad allow-all: `Bash(*)`, a bare `Bash`, or `defaultMode` set to `bypassPermissions` | Claude Code in use | Broad permissions let an agent run anything without asking. |
| +1 | Hooks are configured in `.claude/settings.json` | Claude Code in use | Hooks enforce the standard automatically, for example by running tests before the agent finishes. |

A repository with no config files scores a real 0, never insufficient data: the first three criteria always apply, so at least 3 points do.
*Why:* a missing setup is exactly what this dimension measures.

When Claude Code is in use and `.claude/settings.json` is not committed, the permissions and hooks criteria score 0, not "not applicable".
*Why:* a team that has a `CLAUDE.md` but runs Claude Code with no committed settings, often pushing directly with no trailers, is exactly the team missing permissions and hooks.

## Test discipline (5 points)

| Points | Criterion | Why |
|---|---|---|
| 0–4 | Among PRs that change source files, the share that also change test files: **80%** or more is 4, **60–79%** is 3, **40–59%** is 2, **20–39%** is 1, below **20%** is 0. | Code that changes without tests is the main risk with AI-generated code. |
| +1 | A workflow runs tests on `pull_request`. | Tests that never run in CI don't protect the main branch. |
| −1 | AI-assisted PRs include tests at least **15** percentage points less often than other PRs. The gap is always reported as a finding when it can be computed. | It shows whether agents are held to the same bar as people. |

The score is floored at 0 and capped at 5.

**What counts as a test file:** files in a test folder (`test/`, `tests/`, `__tests__/`, `spec/`, `e2e/`, `fixtures/`, `testdata/`, `androidTest/`, `cypress/`, `playwright/`, including Maven and Gradle `src/test/**`), and files named like tests: `*.test.*`, `*.spec.*`, `test_*.py`, `*_test.py`, `conftest.py`, `*_test.go`, `*Test.java`, `*Tests.java`, `*IT.java`, `*Test.kt`, `*_spec.rb`. Anything under `src/main/` is source, never a test, whatever its package or class name. Gradle build scripts (`*.gradle.kts`), generated output (`build/`, `out/`, `target/`, `dist/` outside a `src/` tree) and vendored code are neither.
*Why:* Java, Android and Python teams lay out tests differently from JavaScript teams; a test helper or an instrumented test must count, and a build script must not look like untested code.

## Review depth (5 points)

| Points | Criterion | Why |
|---|---|---|
| 0–5 | The share of merged PRs reviewed by someone other than the author: **90%** or more is 5, **75–89%** is 4, **50–74%** is 3, **25–49%** is 2, any but below **25%** is 1, none is 0. | A second person reading the change is the core safeguard. |
| −1 | More than **60%** of qualifying PRs are approved silently. Qualifying PRs are approved, change more than **100** lines, and were not opened by a bot. An approval is silent when, up to and including it, nobody other than the author left a review body, an inline review comment or a PR conversation comment. Comments from bots (deploy previews, coverage reports) are not discussion. Needs at least **5** qualifying PRs, otherwise not applicable. | A silent approval on a substantial change suggests rubber-stamping; on a 10-line fix it is normal, so small PRs don't count. |
| −1 | More than **20%** of large PRs (over **400** changed lines) are approved within **10** minutes of being opened. | Nobody reviews 400 lines properly in 10 minutes. |

The score is floored at 0.

For a single maintainer (see Safety gates), the share reviewed by someone else is not applicable, with the same reason, "Single maintainer: GitHub doesn't allow approving your own PR." The two penalties are unchanged; without reviews they are not applicable anyway, so review depth shows insufficient data rather than 0.
*Why:* there is no one else to review, so a 0 would only measure team size. The self-review checklist fix replaces it.

## PR hygiene (5 points)

| Points | Criterion | Why |
|---|---|---|
| +2 | The median PR changes **200** lines or fewer. | Small PRs get real reviews, and agents tend to produce large ones. |
| +1 | The median PR changes **201–400** lines (instead of the +2 above). | Still reviewable, with effort. |
| +1 | A PR template exists. | A template makes authors say what they changed and why. |
| +1 | The template asks about AI agent use and how the change was verified. | Reviewers need to know what to check more carefully. |
| +1 | At least **80%** of PRs have a non-empty description. | A PR without a description cannot be reviewed against its intent. |

## Safety gates (5 points, one each)

| Criterion | Why |
|---|---|
| The default branch requires reviews (read from rulesets). | Without it, the review standard is optional. |
| The default branch requires status checks. | Without it, failing tests can still be merged. |
| Secret scanning runs in CI, **or** GitHub's built-in secret scanning is enabled. With no CI scanner and the setting unreadable (it needs admin access), this is unknown, not 0. | Agents copy credentials into code more readily than people do. |
| A dependency review or audit step runs in CI. | Agents add dependencies freely; something should check them. |
| A `CODEOWNERS` file exists. | It routes changes to the people who know the code. |

If the token cannot read a criterion, it is marked unknown and excluded from applicable points.

With only classic branch protection, "requires reviews" is unknown, with the reason "Reviews may be required via classic branch protection, which this token can't read." It is never inferred from the reviews PRs actually received.
*Why:* review depth already scores behaviour; this criterion is about the rule, and guessing it would count the same evidence twice.

With fewer than **2** human contributors in the window (people who opened or reviewed a PR; bots excluded), "requires reviews" is not applicable, with the reason "Single maintainer: GitHub doesn't allow approving your own PR." With no human PRs in the window the team size is unknown, and the criterion is scored as usual. No login is ever shown.
*Why:* a single maintainer can't approve their own PR, so requiring reviews would block every merge; CI and a self-review checklist are the gates they can use.

On a private repository whose GitHub plan doesn't enforce branch rules (GitHub Free: rulesets and branch protection need GitHub Pro for a personal account or GitHub Team for an organization), "requires reviews" and "requires status checks" are not applicable, with the reason "Your GitHub plan doesn't enforce branch rules on private repositories (needs GitHub Pro for personal accounts, GitHub Team for organizations)." This takes precedence over the single-maintainer rule. Knitwise recognises it only when the repository is private and the branch rules endpoint answers 403 with a message about upgrading (it mentions "upgrade", "GitHub Pro" or "GitHub Team"). Any other 403 from that endpoint makes both criteria unknown, with the reason "Couldn't read branch rules (permission)". One fix replaces the review and check fixes: "Enforce PRs and passing checks: needs a paid GitHub plan (Pro for a personal account, Team for an organization)". Its second line is labelled "Until then:" and points to the direct-push observation for a team, or a self-review checklist for a single maintainer. The Safety gates details then say "Required reviews and required checks can't be enforced on this plan; the score covers the other gates.", and the overall level is at most 4 (see Overall level). `CODEOWNERS` is still scored on whether the file exists, and its evidence adds "On this plan, CODEOWNERS documents ownership but doesn't request reviews."
*Why:* a team on GitHub Free can't turn these rules on without paying, and a ruleset they import is silently not enforced. Scoring 0 would read as neglect, and scoring them met would claim protection that isn't there.

To have these settings read (classic required approvals, built-in secret scanning), pass the Action's optional `admin-token` input: a token with read access to repository administration only, used for those reads and nothing else.

Whether secret scanning **push protection** is on is reported as a finding, not scored.

The Safety gates details always list unknown criteria with their reason. The report also suggests re-running with `admin-token` (a note in the Safety gates details and a line in the footer) only when those unknown criteria could change the **overall level**: that is, if counting them all as met, or all as not met, would give a different level. *Why:* a note on every report would be noise; it is worth a re-run only when the answer could change the headline.

New dependencies added in AI-assisted PRs are also listed as a finding for manual review. This is not scored.
*Why:* the Action may call only api.github.com (NFR-2), so it cannot check package registries itself. A person should confirm each new package is the one intended.

## Adoption signal (5 points)

This scores how **visible and even** AI adoption is, not how much AI the team uses.

| Points | Criterion | Why |
|---|---|---|
| 0–2 | The share of PRs that **record** AI use in a co-author trailer, a label or the PR description: **5%** or more is 1, **20%** or more is 2. Either needs at least **3** recorded PRs. | You can't govern what you can't see, and one recorded PR is a one-off, not a habit. |
| +1 | At least **40%** of active contributors have an AI-assisted PR. | Below that, the practice depends on a few people. |
| +1 | At least **70%** do. | The team shares one way of working. |
| +1 | **Recording consistency:** of the PRs detected as AI-assisted by any signal, at least **50%** also record it. Needs at least **5** AI-assisted PRs. | Agent use that shows up only in branch names or bot authors isn't something the team chose to make visible. |

All four criteria are ratios, so with fewer than 10 PRs this dimension is insufficient data. With fewer than **3** contributors, the two contributor-share criteria are not applicable, and only the team total is shown.

How these are read:

- **A record** is a co-author trailer, a label or the PR description (a template answer or an agent's own footer): something the team chooses to leave. Agent branch names and bot authors are not records, though they still count as AI use for the contributor shares and for consistency.
- **Active contributors** are the people who authored merged PRs in the window. Bots are not contributors, so a PR opened by an agent's bot credits no one.
- PRs opened by automation bots (Dependabot, Renovate) are left out of the recorded share.

## Observations (not scored)

Observations appear above Top fixes. **They never affect any score or the overall level**, and adding them didn't change the rubric version.

- **Direct pushes:** the Action counts commits on the default branch in the lookback window, and how many have a merged pull request. With at least **10** commits and **more than 50%** pushed without a pull request, the report says: "{direct} of {total} commits on {branch} ({pct}%) were pushed directly, without a pull request. Reviews, CI gates and AI-use records only cover changes that go through PRs." The **300** most recent commits are read by default (the optional `max-commits` input allows up to **1000**); with more in the window, the counts cover those and the wording becomes "{direct} of the most recent {n} commits on {branch} …". Only the counts are read (no commit IDs, messages or authors), and they appear in `score.json` under `observations`, with `sampled` and `sampleSize`. If the commits can't be read, the observation is left out and the run carries on.

*Why:* the dimensions above only see pull requests. When most changes skip them, a good score or "insufficient data" says little about how the team really works.

## Top fixes

The report lists up to **3** fixes, chosen from criteria that scored below their maximum in dimensions that have a score. Not-applicable and unknown criteria are never fixes, with one exception: for a single maintainer, "Require CI to pass before merging, and use a self-review checklist in your PR template" (effort S, critical) takes the place of required reviews (safety gates) and of the review-depth fix. It appears once, even when review depth has no score, and the separate required-status-checks fix is folded into it. They are ranked:

1. **Critical first:** broad allow-all agent permissions; no required reviews or status checks on the default branch; AI-assisted PRs including tests at least 15 percentage points less often; no secret scanning of any kind.
2. **Then by score points recoverable,** scaled like the dimension, so a point in a dimension with 3 applicable points counts more than one in a dimension with 5.
3. **Then by effort:** configuration-file changes (S) before CI changes (M) before habit changes (L).

*Why:* the riskiest gaps should lead even when a bigger but safer gain is available; among equals, the cheapest fix first.

Each fix appears once. Criteria that share a fix (for example the two "spread the practice" criteria) are listed as one fix that recovers the points of all of them, and a fix whose setup change is part of another's ("Record AI use consistently" within "Record AI use on PRs") is folded into it. Duplicates are removed before the top 3 are chosen.
*Why:* the same advice twice wastes one of only three places.

## Overall level (1–5)

Average the scores of the dimensions that have data, then map the average to a level:

| Average | Level |
|---|---|
| below 1.0 | 1 |
| 1.0–1.9 | 2 |
| 2.0–2.9 | 3 |
| 3.0–3.9 | 4 |
| 4.0 or more | 5 |

If fewer than **4** of the 6 dimensions have data, the overall level is insufficient data.
*Why:* a level built from two or three dimensions would overstate what we know.

If **the GitHub plan doesn't enforce branch rules** on a private repository (the same detection as in Safety gates), the overall level is **at most 4**, whether or not safety gates has a score. The report shows "Overall level capped at 4: your GitHub plan doesn't enforce branch rules on this private repository." under the level, only when the cap lowered it. Nothing else caps the level.
*Why:* level 5 means the practice is enforced. Without required reviews and required checks, it can't be, however well the other gates and dimensions score.

## Alignment with industry frameworks

| Area | [DORA AI Capabilities Model (2025)](https://dora.dev/ai/capabilities-model/) | [OpenSSF Scorecard](https://scorecard.dev) checks |
|---|---|---|
| Agent configuration | Clear and communicated AI stance | — |
| Test discipline | — | CI-Tests |
| Review depth | Strong version control practices | Code-Review |
| PR hygiene | Working in small batches | — |
| Safety gates | Strong version control practices | Branch-Protection, CI-Tests; SAST and dependency checks (Vulnerabilities, Dependency-Update-Tool) for the secret-scanning and dependency criteria |
| Adoption signal | Clear and communicated AI stance | — |
| Direct-push observation (not scored) | Strong version control practices; Working in small batches | Branch-Protection, Code-Review |

This rubric is informed by the [DORA AI Capabilities Model](https://dora.dev/ai/capabilities-model/) (capability names as in its [survey questions](https://dora.dev/ai/capabilities-model/questions/)) and the [OpenSSF Scorecard](https://github.com/ossf/scorecard) project, and each area relates to the parts shown; a Knitwise score is not a DORA or Scorecard result. The other four DORA AI capabilities (Healthy data ecosystems, AI-accessible internal data, User-centric focus and Quality internal platform) are organisation-level and can't be observed from a repository, so Knitwise doesn't score them.

## Changes

**0.2** (Knitwise Assess 0.1.1), from 0.1:

- **Command recognition:** commands count if they exist in a Maven, Gradle, Go or Cargo build file, in a `package.json` up to 3 folders deep, or as a script in the repository. 0.1 checked only the root `package.json`, `Makefile` and `pyproject.toml`.
- **Claude Code detection:** a committed `CLAUDE.md` or `.claude/` folder means Claude Code is in use, whatever the PRs show, so a missing `.claude/settings.json` scores 0 instead of "not applicable".
- **Single-maintainer rule (safety gates):** with fewer than 2 human contributors, "requires reviews" is not applicable, and the fix "Require CI to pass before merging, and use a self-review checklist in your PR template" takes its place.
- **Single-maintainer rule (review depth):** "reviewed by someone other than the author" is not applicable for a single maintainer, with the same reason.
- **Top fixes:** each fix appears once.

These change which criteria apply and how many points a repository can earn, so **scores from rubric 0.1 and 0.2 shouldn't be compared directly.** Re-run the Action to get a 0.2 baseline.

**0.2.1** (Knitwise Assess 0.1.3), from 0.2. Correction: test-file and source-file classification; no criteria or thresholds changed; scores may shift for Java, Kotlin, Android and Python repos.

- **Test-file detection:** Android `androidTest/` folders, `cypress/`, `playwright/` and `conftest.py` now count as tests. Code under `src/main/` always counts as source, even in a package named `build`, `out`, `target`, `spec`, `test` or `fixtures`. Gradle `*.gradle.kts` build scripts no longer count as source. Test discipline can change for Java, Kotlin, Android and Python repositories.

**0.2.2** (Knitwise Assess 0.1.4), from 0.2.1. Correction for private repositories on GitHub Free; one new threshold (the level cap of 4), no existing thresholds changed.

- **Plan-limited branch rules (safety gates):** Branch rules your GitHub plan doesn't enforce on private repositories are not applicable, not 0. Required reviews and required status checks drop out of the applicable points, so a private repository on GitHub Free is scored on secret scanning, dependency review and `CODEOWNERS` only. With fewer than 3 applicable points, safety gates has insufficient data.
- **Overall level cap:** when this plan limit applies, the overall level is at most 4, whether or not safety gates has a score, with a note under the level when the cap lowers it. Level 5 means the practice is enforced.
