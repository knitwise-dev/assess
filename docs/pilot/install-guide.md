# Install guide

Installing the Knitwise by Blore.AI assessment, for the engineering lead. Takes under 5 minutes. You need permission to add a workflow file to the repository.

## 1. Add the workflow

Create `.github/workflows/ai-practice-assessment.yml` with:

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
      # Replace the placeholder with the release's commit SHA (see below).
      - uses: knitwise-dev/assess@<FULL-COMMIT-SHA-OF-v0.1.4> # v0.1.4
        with:
          lookback-days: 90
      # Optional: keeps report.md and score.json as a downloadable artifact.
      - uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1
        with:
          name: ai-practice-report
          path: ai-practice-report
          retention-days: 30
```

The permissions are read-only. The Action can't change your code, settings, issues or pull requests.

**Pin to a release (recommended).** Replace `<FULL-COMMIT-SHA-OF-v0.1.4>` with the 40-character commit SHA of the v0.1.4 tag. To get it, run `git ls-remote https://github.com/knitwise-dev/assess v0.1.4`, or open the [tags page](https://github.com/knitwise-dev/assess/tags). Keep the `# v0.1.4` comment, so reviewers and Dependabot (`github-actions` ecosystem) can tell which release it is and propose updates.

**Simpler: `@v0`.** Write `uses: knitwise-dev/assess@v0` instead, and every 0.x release is used automatically.

Trade-off: a pinned SHA is reviewable and can't change under you; `@v0` gets fixes without a PR, but also changes without one.

Commit the file to your default branch.

## 2. Run it

1. Open the repository's **Actions** tab.
2. Select **AI practice assessment** in the list on the left.
3. Click **Run workflow**, keep the default branch, and click **Run workflow** again.

A run usually takes under 2 minutes; very busy repositories can take a few minutes.

> TODO: screenshot of the Actions tab with "Run workflow" highlighted.

## 3. Read the report

Open the finished run. The report is on the run's **Summary** page: the overall level, a table of six scores, the top fixes, and a section per dimension explaining each score.

> TODO: screenshot of the job summary showing the report.

## 4. Download the files (optional)

If you kept the upload step, scroll to **Artifacts** at the bottom of the run's Summary page and download **ai-practice-report**. It contains:

- `report.md`: the same report as the summary.
- `score.json`: scores and counts only. No code and no names.

The artifact is kept for 30 days (the `retention-days` above).

> TODO: screenshot of the Artifacts section.

## Optional inputs

Add any of these under `with:`.

| Input | What it does |
|---|---|
| `lookback-days` | How many days of merged PRs to look at. Default 90. If the report says there weren't enough PRs, use the larger value it suggests; if it says the window already covers the full history, a longer one won't help. |
| `max-prs` | The most PRs to analyse. Default 300. |
| `max-commits` | How many recent commits on the default branch to check for direct pushes. Default 300, at most 1000. A larger sample adds about 4 seconds per 25 commits on busy repositories. |
| `exclude-paths` | Extra paths to leave out of PR size, comma-separated, for example `docs/**,*.csv`. Lockfiles, generated files and test data are already left out. |
| `show-cta` | Set to `false` to leave the "What's next" offer out of the report. |
| `admin-token` | Lets the Action read two settings the default token can't see. Not required. See [security-faq.md](security-faq.md). |

## Remove it

Delete `.github/workflows/ai-practice-assessment.yml` and commit. Nothing else is installed. Past runs and artifacts stay in the Actions tab until GitHub's retention removes them, or you can delete them there.
