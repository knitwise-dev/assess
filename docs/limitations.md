# Limitations

What the Knitwise by Blore.AI Assess Action (`knitwise-dev/assess`) can't do, or does only partly. For each, what it means for you and what you can do about it. Scoring details are in [scoring-rubric.md](scoring-rubric.md); what it reads and keeps is in the [security FAQ](pilot/security-faq.md).

## Scope

- **GitHub only.** The Action reads GitHub's API: there's no GitLab, Bitbucket or Azure DevOps support. It is built and tested against GitHub.com; GitHub Enterprise Server is untested.
  *Workaround:* none today.
- **Runs in GitHub Actions, on Node 24.** The Action declares `runs.using: node24`. GitHub-hosted runners support it; a self-hosted runner must be recent enough to run Node 24 actions.
  *Workaround:* on self-hosted runners, update the runner before installing.
- **One repository per run.** A report covers the repository the workflow runs in. There's no organisation-wide view.
  *Workaround:* install the workflow in each repository you want assessed and compare the reports yourself.
- **Configuration is read from the default branch.** Agent instructions, settings, workflows and build files are read as they are on the default branch now, not as they were during the lookback window, and not from other branches.
  *What it means:* a fix you merged yesterday counts today, even though the PRs in the window were made before it.
- **Only merged pull requests in the window are read:** 90 days by default, at most 300 PRs (`max-prs`). Open and closed-unmerged PRs are ignored. When the cap is reached, the report says so and covers the most recent ones.
  *Workaround:* `lookback-days` and `max-prs` change the window.

## What it can't see

- **Changes that never go through a pull request.** Tests, reviews and AI-use records are read from merged PRs only. Commits pushed straight to the default branch are only *counted*, for the unscored direct-push observation, which appears above the top fixes when more than half of at least 10 commits in the window skipped PRs.
  *What it means:* when most changes skip PRs, the scores describe a minority of the work.
- **AI use that isn't recorded.** Detection is best-effort: co-author trailers, agent branch names (`claude/`, `codex/`), agent bot authors, labels (an agent's name, or `ai`, `ai-assisted`, `ai-generated`, `ai-agent`, `llm-generated`) and PR descriptions (agent footers and links, or an AI answer in the PR template). Agent use that leaves none of these is invisible, so AI-assisted counts are a floor.
  *Workaround:* record AI use consistently, for example a `Co-authored-by` trailer or a PR template question; the Adoption dimension rewards that.
- **Agents other than Claude Code and Codex aren't assessed.** Agent configuration checks `CLAUDE.md`, `AGENTS.md` and `.claude/settings.json` only. Cursor rules, Copilot instructions, Gemini and other tools' files aren't read or scored. A PR labelled `ai-assisted` still counts as AI-assisted, without an agent name.
  *What it means:* a team that uses only Cursor or Copilot will score low on Agent configuration whatever its setup.
- **Outcomes.** It doesn't measure reverts, CI failure rates, lead time or code quality, and it isn't a DORA metrics tool, a security scanner or a code reviewer. Safety gates check that a secret scanner and a dependency check *run*, not what they find.
- **Reviews and checks outside GitHub's PR records.** Reviews done elsewhere (pairing, another review tool) don't count. Secret scanners and dependency checks are recognised by name in your workflows (for example TruffleHog, gitleaks, OSV-Scanner, GitHub's dependency review, `npm audit`); a custom scanner isn't recognised.
  *Workaround:* the report shows the evidence for each criterion; treat a "not met" for a tool it doesn't know as a false negative.
- **Commands only in code.** Build, test and lint commands are read from inline code and code blocks in `CLAUDE.md` and `AGENTS.md`, not from prose. `.claude/settings.local.json` is personal and ignored.

## Data needs

- **At least 10 merged PRs** in the window for any ratio (test share, review share, PR size, adoption). Below that, those criteria are not applicable and most dimensions show "insufficient data".
- **At least 3 applicable points** for a dimension to be scored.
- **At least 4 of the 6 dimensions** with data for an overall level.
- **Comparisons of AI-assisted and other PRs** need at least 5 PRs in each group; **per-contributor figures** need at least 3 contributors.

*What it means:* small or new repositories often get "insufficient data". That's deliberate: percentages over a handful of PRs mislead.
*Workaround:* the report suggests a longer `lookback-days` when that would help, and says when the window already covers the repository's whole history.

## Heuristics

- **Test and source files are told apart by path.** Test folders: `test/`, `tests/`, `__tests__/`, `spec/`, `e2e/`, `fixtures/`, `testdata/`, `androidTest/`, `cypress/`, `playwright/` (including Maven and Gradle `src/test/**`). Test file names: `*.test.*`, `*.spec.*`, `test_*.py`, `*_test.py`, `conftest.py`, `*_test.go`, `*Test.java`, `*Tests.java`, `*IT.java`, `*Test.kt`, `*_spec.rb`. Anything under `src/main/` is source. Gradle build scripts, build output and vendored code are neither.
  *What it means:* a layout outside these patterns can be miscounted.
- **Build commands are recognised for these ecosystems:** npm, pnpm, Yarn and Bun scripts in any `package.json` up to 3 folders deep; root `Makefile` targets; root `pyproject.toml` scripts and tools; Maven (`mvn`, `mvnw`) with a `pom.xml`; Gradle (`gradle`, `gradlew`) with a `build.gradle(.kts)`; `go` with a `go.mod`; `cargo` with a `Cargo.toml`; and scripts committed in the repository (`*.sh`, `mvnw`, `gradlew`). Other build systems (Bazel, .NET, CMake and others) aren't recognised.
- **Changed lines exclude lockfiles, generated files and test data.** Lockfiles: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `poetry.lock`, `Pipfile.lock`, `Cargo.lock`, `go.sum`. Generated: files your `.gitattributes` marks `linguist-generated`, minified files and source maps. Test data: snapshots and fixture folders (full list in the rubric).
  *Workaround:* `exclude-paths` leaves out more, and `linguist-generated` in `.gitattributes` marks your own generated files.

## People

- **Team-level only.** No developer is named, ranked or scored; logins are replaced with anonymous ids as they're read. With one or two contributors, though, team figures necessarily describe those people.
- **Accounts, not people.** One person with two GitHub accounts counts as two contributors.
- **Bots are excluded** from contributor counts and from reviews. A bot is an account GitHub marks as a bot (for example Dependabot); a bot running under a normal user account counts as a person.
- **Single maintainer.** With fewer than 2 human contributors (people who opened or reviewed a PR), "reviewed by someone else" and "requires reviews" are not applicable: GitHub doesn't let you approve your own PR.

## GitHub plan limits

- **Private repositories on GitHub Free:** GitHub doesn't enforce rulesets or branch protection there, and won't return them to the API. Required reviews and required checks are then not applicable, and the overall level is capped at 4: level 5 means the practice is enforced.
  *Workaround:* GitHub Pro (personal account) or GitHub Team (organisation) enforces them.
- **Settings the default token can't read:** how many approvals classic branch protection requires, and whether GitHub's built-in secret scanning and push protection are on. These are marked unknown, never guessed or counted as 0.
  *Workaround:* the optional `admin-token` input, a token with read-only repository administration access, used for those reads only.

## Sampling and runtime

- **The direct-push observation reads the most recent 300 commits** on the default branch in the window (up to 1000 with `max-commits`); past that, it reports a sample.
- **Run time:** usually under 2 minutes; very busy repositories can take a few minutes, and a larger `max-commits` adds about 4 seconds per 25 commits.
- **API rate limits:** the Action waits and retries twice. If the direct-push observation or a setting still can't be read, that part is left out with a reason and the run continues. If the merged pull requests or the repository's file list can't be read, the run fails, because there would be nothing reliable to score.
  *Workaround:* re-run later.

## Scores

- **Compare scores only within one rubric version.** Every `score.json` records its `rubricVersion`; criteria and thresholds change between versions (see "Changes" in the rubric).
- **All dimensions weigh equally** in the overall level, which is the average of the dimensions with data.
- **Informed by DORA and OpenSSF Scorecard, not certified by them.** A Knitwise score is not a DORA or Scorecard result.

## Over time

- **Each run is a snapshot.** There's no history or trend view, and nothing is stored between runs except what you keep (the job summary and the optional artifact). A weekly drift digest is planned, not available.
  *Workaround:* keep each run's `score.json` and compare runs with the same rubric version.

## Privacy note

The report goes to the run's job summary and, if you keep the upload step, an artifact: anyone who can read the repository's Actions runs can read it. It contains counts, paths, check names and PR links, never code or developer names, but it does name the repository.
