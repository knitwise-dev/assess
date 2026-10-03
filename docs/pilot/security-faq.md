# Security FAQ

For the person who approves the Knitwise by Blore.AI Assess Action (`knitwise-dev/assess`) as a third-party GitHub Action. One short answer per question.

## What does it read?

From the repository it runs in, through the GitHub API:

- **Merged pull requests** from the lookback window (90 days by default): branch name, labels, description, size, the list of changed files with line counts, review states and times, and who commented when.
- **Commits** in those PRs: only the `Co-authored-by` lines are kept, to detect AI agents.
- **Configuration files:** `CLAUDE.md`, `AGENTS.md`, `.claude/settings.json`, PR templates, workflow files, `CODEOWNERS`, `.gitattributes`, the root `Makefile` or `pyproject.toml`, and build files up to 3 folders deep (`package.json`, `pom.xml`, `build.gradle`, `build.gradle.kts`, `go.mod`, `Cargo.toml`; never inside `node_modules`). Script files (`*.sh`, `mvnw`, `gradlew`) are listed by path only, not read.
- **Commits on the default branch** in the lookback window, the most recent 300 by default (up to 1000 with `max-commits`): only whether each one has a merged pull request. No commit IDs, messages or authors are requested; only the three counts are kept.
- **Dependency changes:** for PRs that change a dependency manifest, the added package names.
- **Repository settings:** the default branch, its protection rules and rulesets.
- **History start:** whether the default branch has any commit older than the lookback window (the date of at most one commit, used only to word the "not enough PRs" note; nothing about it is kept).

## What does it never read or keep?

- **Never read:** source code files other than the configuration files above, diffs other than dependency manifests, and comment text.
- **Read on the runner, never kept:** review text and full commit messages are fetched but only "is there text?" and the co-author lines are kept. PR descriptions are used on the runner for checks and are not copied into the report.
- **Logins** are replaced with anonymous ids as soon as they are read.

## Does it send data anywhere?

No. The Action's own code calls only `api.github.com`, to read. Its REST calls are `GET` only (the client refuses anything else), and its GraphQL calls are read-only queries. If you keep the optional upload step, GitHub's official `actions/upload-artifact` stores the files in your own repository's artifacts. That is GitHub's action, not ours.

## What does it write, and where?

Two files on the runner, in `ai-practice-report/` by default: `report.md` and `score.json`. It also adds the report to the run's job summary. It writes nothing to your repository: no commits, comments, labels, issues or settings.

## How long is it kept?

The job summary stays with the workflow run, for as long as your repository keeps run logs (GitHub's default is 90 days). The artifact is kept for the `retention-days` in your workflow; the install guide uses 30 days. You can delete runs and artifacts at any time from the Actions tab.

## What permissions does it need, and why?

- `contents: read`: to read configuration files and repository settings.
- `pull-requests: read`: to read merged pull requests and their reviews.

Nothing else. It works with the run's default `GITHUB_TOKEN`.

## What does the optional admin-token add? Is it required?

It is never required. GitHub hides two kinds of setting from the default token: the approvals required by classic branch protection, and whether built-in secret scanning and push protection are on. An `admin-token` with only *Administration: Read-only* access on the repository lets the Action read those two things, and nothing else. Without it, those criteria are marked unknown and left out of the score; they are never counted as 0.

## Does it rank or name developers?

No. Reports aggregate to team and repository level, and nobody is named, ranked or scored. One exception: the report shows the repository name, so for a repository under a personal account (`your-account/repo`) it includes that account name.

## How can we audit the code?

- **Source:** the Action's source is published at `knitwise-dev/assess` under the Apache License 2.0 before teams outside Blore.AI are asked to run it. Each scoring rule is documented in `docs/scoring-rubric.md`.
- **Pinned versions:** npm dependencies are pinned to exact versions, and every action in our workflows is pinned to a full commit SHA.
- **Built from source:** the Action runs `dist/index.js`, which is built from the source and committed. CI rebuilds it on every change and fails if the committed file differs.

You can also read the report and `score.json` from your first run before sharing anything.

## What do we share with Blore.AI?

Nothing by default. The Action sends us nothing. If you want our help, share `score.json` only: it holds scores, points and criterion results, and the commit counts behind the direct-push observation, never code or names. The report's "What's next" section can be turned off with `show-cta: false`.
