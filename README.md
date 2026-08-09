# gh-actions

Shared, reusable GitHub Actions for Node/Bun projects — write the logic once here, reference it from any project.

Repo: `Anparasan3/gh-actions`

## What's inside

| Name | Path | Type | Purpose |
|---|---|---|---|
| Setup Bun | `setup-bun/` | composite action | Installs Bun, caches deps, `bun install` |
| Lint | `lint/` | composite action | Runs your lint script |
| Test | `test/` | composite action | Runs your test script |
| Build | `build/` | composite action | Runs your build script |
| Release (version calc) | `release/` | composite action | Calculates the next semver tag from a bump type |
| PR Checks | `.github/workflows/pr-checks.yml` | **reusable workflow** | Jira link + lint + test + coverage + deploy + healthcheck, fully toggleable |

`example-ci-workflow.yml`, `release/example-workflow.yml`, `pr-checks-example-caller.yml`, and `example-dependabot.yml` are **not actions** — they're files you copy into each project's `.github/` folder.

## Versioning — pin your usage

Tag releases in **this** repo, then pin every calling project to a tag so changes here don't silently ripple into every project the moment you push to `main`:

```bash
git tag v0.1.0
git push origin v0.1.0
```

Then in projects:
```yaml
uses: Anparasan3/gh-actions/setup-bun@v0.1.0
```

Bump the tag (`v0.1.1`, `v0.2.0`, ...) when you're ready to roll out changes deliberately, and update the version string in each project when you want it to pick up the change.

## Quick start — CI in your project

Copy `example-ci-workflow.yml` into `your-project/.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: Anparasan3/gh-actions/setup-bun@v0.1.0
      - uses: Anparasan3/gh-actions/lint@v0.1.0
      - uses: Anparasan3/gh-actions/test@v0.1.0
      - uses: Anparasan3/gh-actions/build@v0.1.0
```

### Action inputs (setup-bun / lint / test / build)

```yaml
- uses: Anparasan3/gh-actions/setup-bun@v0.1.0
  with:
    bun-version: latest        # optional, default: latest
    working-directory: .       # optional, default: repo root
    install: 'true'            # optional, set 'false' to skip bun install

- uses: Anparasan3/gh-actions/lint@v0.1.0    # same shape for test/build
  with:
    working-directory: .
    script: lint                # override if your package.json script differs
```

## Quick start — Release in your project

Copy `release/example-workflow.yml` into `your-project/.github/workflows/release.yml`. It:

- Runs on manual dispatch (`workflow_dispatch`) with a `patch`/`minor`/`major` choice → calculates the next tag via the shared `release` action, pushes it, creates a GitHub Release.
- Runs on any `v*` tag pushed directly → just creates the GitHub Release for that tag.

## Quick start — PR Checks in your project

This is a **reusable workflow**, not a composite action, since it needs its own triggers/jobs/secrets. Copy `pr-checks-example-caller.yml` into `your-project/.github/workflows/pr-checks.yml` and configure it per project:

```yaml
name: Pull Request Checks

on:
  pull_request:
    types: [opened, edited, synchronize]

jobs:
  checks:
    uses: Anparasan3/gh-actions/.github/workflows/pr-checks.yml@v0.1.0
    with:
      enable-jira-link: true
      enable-coverage: true
      enable-deploy: true
      enable-healthcheck: true
      lint-script: lint
      test-script: test
      deploy-script: deploy:staging
      healthcheck-url: https://your-app.example.com/healthcheck
      ticket-prefix: DEV
    secrets:
      JIRA_EMAIL: ${{ secrets.JIRA_EMAIL }}
      JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}
      JIRA_DOMAIN: ${{ secrets.JIRA_DOMAIN }}
      AWS_ACCESS_KEY_ID: ${{ secrets.NONPROD_AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.NONPROD_AWS_SECRET_ACCESS_KEY }}
      DEPLOY_ENV_VARS: ${{ secrets.DEPLOY_ENV_VARS }}
```

### Toggles (all default to `false` except where noted)

| Input | Default | What it controls |
|---|---|---|
| `enable-jira-link` | `false` | Requires a `TICKET_PREFIX-123` style ticket in the PR title/body (blocks non-draft PRs without one) and links the PR to that Jira ticket |
| `enable-coverage` | `false` | Posts a coverage report as a PR comment via `romeovs/lcov-reporter-action` |
| `enable-deploy` | `false` | Runs a deploy script after tests pass |
| `enable-healthcheck` | `false` | Curls a healthcheck URL after deploy — **only runs if `enable-deploy` is also `true`** |

Lint and test always run — there's no toggle for those, since skipping them defeats the purpose of a PR-checks workflow.

### Config inputs

| Input | Default | Purpose |
|---|---|---|
| `bun-version` | `latest` | Bun version to install |
| `lint-script` | `lint` | `package.json` script for lint |
| `test-script` | `test` | `package.json` script for tests |
| `deploy-script` | `deploy:staging` | `package.json` script for deploy |
| `healthcheck-url` | `''` | URL to curl after deploy; expects JSON with `"status": "ok"` |
| `aws-region` | `us-east-1` | Passed as `AWS_REGION` env var to the deploy step |
| `ticket-prefix` | `DEV` | Ticket key prefix to search for, e.g. `DEV` matches `DEV-123` (case-insensitive) |

### Secrets

All secrets are optional at the workflow level — only pass the ones your enabled toggles actually need:

| Secret | Needed when |
|---|---|
| `JIRA_EMAIL`, `JIRA_API_TOKEN`, `JIRA_DOMAIN` | `enable-jira-link: true` |
| `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | `enable-deploy: true` (if your deploy script needs AWS creds) |
| `DEPLOY_ENV_VARS` | Optional, `enable-deploy: true` — a JSON string of extra env vars to inject before the deploy step, e.g. `{"MAGENTO_DATABASE_URL":"...","ACCOUNT_DATABASE_URL":"..."}` |

`GITHUB_TOKEN` is handled automatically — no need to pass it explicitly.

## Dependabot

Dependabot config **cannot** be shared via `uses:` — it must live in each project's own `.github/dependabot.yml`. Copy `example-dependabot.yml` into every project that should get automated dependency PRs. It covers:

- **`bun`** ecosystem — updates `package.json` / `bun.lock` (requires a committed lockfile)
- **`github-actions`** ecosystem — updates versioned `uses:` references, including bumping `Anparasan3/gh-actions@v0.1.0` itself once you cut new tags

## Notes

- This repo must be **public** (or the calling repos need extra token/permission setup) — composite actions and reusable workflows referenced cross-repo via `uses:` need read access to this repo.
- `actions/checkout` must always run before any composite action step that operates on the checked-out code.
- Reusable workflows (`pr-checks.yml`) are referenced with the full path including `.github/workflows/`, unlike composite actions which reference a folder containing `action.yml`. Don't mix up the two `uses:` formats.
