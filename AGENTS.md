# AGENTS.md

## What this repo is

Shared, reusable GitHub Actions for Bun projects (repo: `Anparasan3/gh-actions`). Other
repos consume this one via `uses: Anparasan3/gh-actions[/<action>]@<tag>` — there is no
build/install step here, the files themselves are the product.

## Layout

- `action.yml` (repo root) — the main composite action: setup Bun, install, lint, test,
  build, each stage toggleable via `enable-lint`/`enable-test`/`enable-build` inputs.
  Referenced with no subfolder: `Anparasan3/gh-actions@v0.1.0`.
- `release/`, `dependabot/` — smaller single-purpose composite actions, each an
  `action.yml` referenced by subfolder (e.g. `Anparasan3/gh-actions/release@v0.1.0`).
- `.github/workflows/pr-checks.yml` — the one **reusable workflow** (not a composite
  action), referenced with the full `.github/workflows/` path.
- `.github/workflows/self-test.yml` — this repo's own CI: actionlint over everything,
  a fixture-based smoke test of the root action's toggles, and an assert-based check
  of the `release` action's version-bump math. Run/update this when touching either.
- `example-ci-workflow.yml`, `release/example-workflow.yml`,
  `pr-checks-example-caller.yml`, `example-dependabot.yml` — not actions, just templates
  meant to be copied into a *calling* project's `.github/`.

## Working in this repo

- Editing an `action.yml` changes behavior for every project pinned to that tag once
  they bump their pin — check `self-test.yml` still passes and that README.md's
  documented inputs/defaults still match.
- If you add/rename an input, output, or default, update the corresponding table in
  `README.md` in the same change — it's the only docs and projects copy from it.
- Versioning is tag-based (`v0.1.0`, ...). Don't assume `main`/`master` is what
  consumers run; changes only take effect for a project when it bumps its pinned tag.
- Keep the example/template files (`example-*.yml`, `pr-checks-example-caller.yml`)
  in sync with `pr-checks.yml`'s actual inputs/secrets — they're copy-paste sources,
  not executed here, so nothing else will catch drift.
