# AGENTS.md

## What this repo is

Shared, reusable GitHub Actions for Bun projects (repo: `Anparasan3/gh-actions`). Other
repos consume this one via `uses: Anparasan3/gh-actions/<action>@<tag>` — there is no
build/install step here, the files themselves are the product.

## Layout

- `setup-bun/`, `lint/`, `test/`, `build/`, `release/` — composite actions, each an
  `action.yml` (inputs/outputs + `runs: using: composite`). No other files per action.
- `.github/workflows/pr-checks.yml` — the one **reusable workflow** (not a composite
  action), referenced with the full `.github/workflows/` path.
- `example-ci-workflow.yml`, `release/example-workflow.yml`,
  `pr-checks-example-caller.yml`, `example-dependabot.yml` — not actions, just templates
  meant to be copied into a *calling* project's `.github/`.

## Working in this repo

- Editing an `action.yml` changes behavior for every project pinned to that tag once
  they bump their pin — there's no test suite here, so read the composite steps
  carefully and check README.md's documented inputs/defaults still match.
- If you add/rename an input, output, or default, update the corresponding table in
  `README.md` in the same change — it's the only docs and projects copy from it.
- Versioning is tag-based (`v0.1.0`, ...). Don't assume `main` is what consumers run;
  changes only take effect for a project when that project bumps its pinned tag.
- Keep the example/template files (`example-*.yml`, `pr-checks-example-caller.yml`)
  in sync with `pr-checks.yml`'s actual inputs/secrets — they're copy-paste sources,
  not executed here, so nothing else will catch drift.
