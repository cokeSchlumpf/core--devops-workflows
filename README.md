# Core DevOps Workflows

Reusable GitHub Actions workflows for checks, documentation and releases of Python projects built with
[Poetry](https://python-poetry.org/) and [Poe the Poet](https://poethepoet.natn.io/).

| Workflow | Purpose |
| --- | --- |
| `python-check.yml` | Installs the project, runs `poe check` and builds the docs with `poe docs-build` |
| `python-docs.yml` | Builds the docs with `poe docs-build` and deploys them to GitHub Pages |
| `python-release.yml` | Runs [release-please](https://github.com/googleapis/release-please), moves `versions/<major>` to each release and optionally polishes the Release PR notes with an LLM |
| `conventional-pr-title.yml` | Fails if a PR title isn't a [Conventional Commit](https://www.conventionalcommits.org/) |
| `.github/actions/polish-changelog` | Composite action used by `python-release.yml` for the LLM polish |

## Versions

Reference the workflows via their major version branch, e.g. `@versions/1`. It always points to the latest `1.x`
release, so fixes arrive automatically while breaking changes need a new major version. Pin a tag (e.g. `@v1.0.0`) for
fully reproducible builds.

`python-release.yml` loads the polish action from this repository at its `workflows-ref` input (default `versions/1`).
Update that default when releasing a new major version.

## Usage

A project needs:

- The Poe tasks `check` (linting, type checks, tests) and `docs-build` (strict build into `site/`) in `pyproject.toml`.
- `release-please-config.json`, `.release-please-manifest.json` and `CHANGELOG.md` for releases.

Then add the following callers to `.github/workflows/`:

```yaml
# check.yml
name: Check

on:
  push:
    branches:
      - "feature/**"
      - "fix/**"

jobs:
  check:
    uses: cokeSchlumpf/core--devops-workflows/.github/workflows/python-check.yml@versions/1
```

```yaml
# docs.yml
name: Docs

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  docs:
    uses: cokeSchlumpf/core--devops-workflows/.github/workflows/python-docs.yml@versions/1
```

```yaml
# release.yml
name: Release

on:
  push:
    branches:
      - main

permissions:
  contents: write
  issues: write
  pull-requests: write

concurrency:
  group: release
  cancel-in-progress: false

jobs:
  release:
    uses: cokeSchlumpf/core--devops-workflows/.github/workflows/python-release.yml@versions/1
    secrets: inherit
```

```yaml
# pr-title.yml
name: PR Title

on:
  pull_request:
    types: [opened, edited, synchronize, reopened]

permissions:
  pull-requests: read

jobs:
  pr-title:
    uses: cokeSchlumpf/core--devops-workflows/.github/workflows/conventional-pr-title.yml@versions/1
```

### Inputs

| Workflow | Input | Default |
| --- | --- | --- |
| `python-check.yml` | `python-version`, `poetry-version` | `3.12`, `2.1.3` |
| | `install-args` (for `poetry install`) | `--all-extras --with docs` |
| | `docs` (also run `poe docs-build`) | `true` |
| `python-docs.yml` | `python-version`, `poetry-version`, `install-args` | as above |
| | `site-dir` | `site` |
| `python-release.yml` | `config-file`, `manifest-file` | `release-please-config.json`, `.release-please-manifest.json` |
| | `changelog-path`, `openai-model` | `CHANGELOG.md`, `gpt-5-mini` |
| | `workflows-ref` (ref of this repository to load the polish action from) | `versions/1` |
| | secret `OPENAI_API_KEY` (optional, enables the LLM polish) | |
| `conventional-pr-title.yml` | `types` (one per line) | `feat`, `fix`, `perf`, `deps`, `revert`, `docs`, `refactor`, `test`, `build`, `ci`, `chore`, `style` |

### Setting up a repository

Checklist for every repository that calls these workflows. Apart from the files, everything is configured in the
repository's GitHub **Settings**.

- [ ] **Project files:** the Poe tasks `check` and `docs-build`, `release-please-config.json`,
  `.release-please-manifest.json`, `CHANGELOG.md` and the caller workflows from [Usage](#usage).
- [ ] **General → Pull Requests:** only *Allow squash merging* enabled, with *Default commit message* set to
  **Pull request title and description**. Otherwise single-commit PRs use the commit message instead of the PR title,
  and `BREAKING CHANGE:` / `Release-As:` lines in the description don't reach `main`. Recommended: *Automatically delete
  head branches*.
- [ ] **Actions → General → Actions permissions:** if actions are restricted, allow `actions/*`,
  `googleapis/release-please-action`, `amannn/action-semantic-pull-request` and `cokeSchlumpf/core--devops-workflows`.
- [ ] **Actions → General → Workflow permissions:** *Allow GitHub Actions to create and approve pull requests*
  enabled. Otherwise release-please can't open the Release PR.
- [ ] **Pages:** *Source* set to **GitHub Actions** (only with `python-docs.yml`). Otherwise the deploy job fails.
- [ ] **Rules → Rulesets:** a branch ruleset for `versions/*` with *Restrict deletions* and *Block force pushes*. Don't
  enable *Restrict updates*, the release workflow pushes to these branches with the `GITHUB_TOKEN`.
- [ ] **Secrets and variables → Actions:** repository secret `OPENAI_API_KEY` (optional, enables the LLM polish of
  the Release PR notes). Without it, the polish job is skipped.

## Development

```bash
poetry install
poetry run poe check        # linting, type checks and tests (same as CI)
poetry run poe fix          # auto-fix lint issues and format code
```

This repository uses its own workflows via local references (`uses: ./.github/workflows/...`) and is released the same
way as the projects calling it.
