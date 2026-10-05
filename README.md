# Knitwise Assess

**Knitwise by Blore.AI.** Knit your team's AI habits into one practice.

*Pre-release.*

A GitHub Action that assesses how your team governs AI coding agents (Claude Code and Codex). It reads your merged pull requests and repository configuration, then scores six dimensions from 0 to 5: agent configuration, test discipline, review depth, PR hygiene, safety gates and adoption signal. How each score is computed is public: see [`docs/scoring-rubric.md`](docs/scoring-rubric.md).

## Usage

```yaml
name: AI practice assessment
on:
  workflow_dispatch:
permissions:
  contents: read
  pull-requests: read
jobs:
  assess:
    runs-on: ubuntu-latest
    steps:
      - uses: knitwise-dev/assess@v0
        with:
          lookback-days: 90
      # Optional: keeps report.md and score.json as a downloadable artifact.
      - uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1
        with:
          name: ai-practice-report
          path: ai-practice-report
          retention-days: 30
```

The upload step is optional: the report always appears in the run's job summary. The upload keeps `report.md` and `score.json` as a downloadable artifact, uploaded by GitHub's own action rather than by this Action's code.

## Inputs

| Input | Default | Description |
|---|---|---|
| `github-token` | `${{ github.token }}` | Read-only token for the repository. Needs `contents: read` and `pull-requests: read`. |
| `lookback-days` | `90` | How many days of merged PRs to analyse. |
| `max-prs` | `300` | Upper bound on PRs analysed. |
| `max-commits` | `300` | **Optional.** Most recent default-branch commits read for the direct-push observation, 1 to 1000. Invalid values fall back to 300 with a warning. |
| `admin-token` | unset | **Optional.** See below. |
| `output-dir` | `ai-practice-report` | Where `report.md` and `score.json` are written, relative to the workspace. |
| `show-cta` | `true` | Set to `false` to leave the "What's next" section out of the report. |
| `exclude-paths` | unset | **Optional.** Comma-separated globs left out of changed-line counts, on top of lockfiles, generated files and test data (for example `docs/**,*.csv`). |

## Outputs

| Output | Description |
|---|---|
| `overall-level` | Overall maturity level, 1–5, or `insufficient-data`. |
| `report-file` | Path to `report.md`. |
| `score-file` | Path to `score.json`. |

## What this Action can and cannot do

For the person approving this Action:

- **Permissions.** It needs only `contents: read` and `pull-requests: read`. It never writes to your repository: no commits, comments, labels, issues or settings. Its REST client refuses anything but `GET`, and its GraphQL calls are read-only queries.
- **Network.** The Action's own code calls only `api.github.com`. If you add the optional upload step, GitHub's official `actions/upload-artifact` uploads the files, not this Action.
- **What it writes.** Two files in `output-dir` on your runner, `report.md` and `score.json`, and the same report in the run's job summary. Nothing else, and nothing outside the runner.
- **What it reads but never keeps.** Configuration files (such as `CLAUDE.md`, workflows and `package.json`) and the dependency lines of changed manifests are read on your runner to score them. They are not copied into the report or `score.json`.
- **What it never collects.** Source code beyond those configuration files, diffs other than dependency manifests, commit message text (only `Co-authored-by` lines are kept), review and comment text, and developer names or logins. Logins are replaced with anonymous ids as they are read.
- **No individuals.** Reports aggregate to team and repository level. No one is named, ranked or scored. The report shows the repository name. For repos under a personal account, this includes your account name.
- **`score.json`** holds scores, points and criterion statuses only, plus unscored commit counts (total, via PRs, pushed directly). It is the only file we would ever ask you to share.
- **Best-effort AI detection.** AI-assisted PRs are detected from co-author trailers, agent branch names, agent bots, labels and PR descriptions. Agent use that leaves none of these is not detected, and the report says so.
- **Limitations.** What it can't see or measure (for example, changes that skip pull requests, agents other than Claude Code and Codex, small repositories), and the workarounds: [`docs/limitations.md`](docs/limitations.md).

## Optional: `admin-token`

With the default token, two kinds of setting can't be read, because GitHub shows them only with repository administration access, which `GITHUB_TOKEN` never has:

- how many approvals **classic branch protection** requires (rulesets are readable without it);
- whether GitHub's built-in **secret scanning** and **push protection** are enabled.

Without `admin-token`, the criteria that depend on them are marked unknown and left out of the score. They are never guessed and never counted as 0. (A secret scanner in your CI still counts without it.)

To include it, create a token that has **only read access to repository administration**. For example, use a fine-grained personal access token or GitHub App limited to this repository, with the repository permission *Administration: Read-only* and nothing else. Store it as a secret and pass it in:

```yaml
      - uses: knitwise-dev/assess@v0
        with:
          admin-token: ${{ secrets.ASSESS_ADMIN_TOKEN }}
```

The Action uses this token for those two reads (`GET` branch protection and `GET` repository settings) and nothing else. If GitHub still doesn't return a setting for the token, that criterion stays unknown. Avoid classic personal access tokens here: their `repo` scope grants far more than this needs.
