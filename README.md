# Core DevOps Workflows

Reusable GitHub Actions workflows for checks, documentation and releases of Python projects built with
[Poetry](https://python-poetry.org/) and [Poe the Poet](https://poethepoet.natn.io/).

| Workflow | Purpose |
| --- | --- |
| `python-check.yml` | Installs the project, runs `poe check` and builds the docs with `poe docs-build` |
| `python-docs.yml` | Builds the docs with `poe docs-build` and deploys them to GitHub Pages |
| `python-release.yml` | Runs [release-please](https://github.com/googleapis/release-please), moves `versions/<major>` to each release (libraries) and optionally polishes the Release PR notes with an LLM |
| `python-app-deploy.yml` | Deploys a released app version to one stage: builds and pushes its image if missing, applies `deployment/<stage>` with OpenTofu and waits for `/health` to report the version |
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

### Apps

Apps are released like libraries but without a version branch, and the release deploys: the release run calls
`python-app-deploy.yml` in the same run, because a release created with the `GITHUB_TOKEN` triggers no other workflow.
Their `check.yml` and `pr-title.yml` are the same as above (with `python-version: "3.14"` and `docs: false`).

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
  id-token: write

concurrency:
  group: release
  cancel-in-progress: false

jobs:
  release:
    uses: cokeSchlumpf/core--devops-workflows/.github/workflows/python-release.yml@versions/1
    secrets: inherit
    with:
      version-branch: false

  deploy:
    needs: release
    if: ${{ needs.release.outputs.release_created == 'true' }}
    uses: cokeSchlumpf/core--devops-workflows/.github/workflows/python-app-deploy.yml@versions/1
    with:
      environment: uat
      version: ${{ needs.release.outputs.tag_name }}
```

```yaml
# deploy.yml: redeploys an existing tag (after a failed deployment, or to roll back)
name: Deploy

on:
  workflow_dispatch:
    inputs:
      version:
        description: Tag to deploy, e.g. v1.2.0
        required: true

permissions:
  contents: read
  id-token: write

jobs:
  deploy:
    uses: cokeSchlumpf/core--devops-workflows/.github/workflows/python-app-deploy.yml@versions/1
    with:
      environment: uat
      version: ${{ inputs.version }}
```

A further stage (e.g. `dev` on every push to `main`) is another caller with its own `environment` and a folder
`deployment/<stage>/`; deployments to the same stage never run in parallel (`concurrency: deploy-<stage>`).

The app needs a `Dockerfile` with `ARG APP__VERSION`, a `GET /health` answering `{"version": "<version>"}`, and per
stage an OpenTofu root `deployment/<stage>/` with an `azurerm` backend, a variable `image_tag` passed to `sandbox_app`
of [core--terraform-modules](https://github.com/cokeSchlumpf/core--terraform-modules) (`app_name` equal to the
repository name) and an output `app_url`.

### Inputs

| Workflow | Input | Default |
| --- | --- | --- |
| `python-check.yml` | `python-version`, `poetry-version` | `3.12`, `2.5.1` |
| | `install-args` (for `poetry install`) | `--all-extras --with docs` |
| | `docs` (also run `poe docs-build`) | `true` |
| `python-docs.yml` | `python-version`, `poetry-version`, `install-args` | as above |
| | `site-dir` | `site` |
| `python-release.yml` | `config-file`, `manifest-file` | `release-please-config.json`, `.release-please-manifest.json` |
| | `changelog-path`, `openai-model` | `CHANGELOG.md`, `gpt-5-mini` |
| | `workflows-ref` (ref of this repository to load the polish action from) | `versions/1` |
| | `version-branch` (move `versions/<major>` to each release) | `true` |
| | secret `OPENAI_API_KEY` (optional, enables the LLM polish) | |
| `python-app-deploy.yml` | `environment` (GitHub environment and stage), `version` (tag, e.g. `v1.2.0`) | required |
| | `app-name` (image name, equal to `app_name` of the deployment) | repository name |
| | `deployment-dir` (OpenTofu root) | `deployment/<environment>` |
| | environment variables `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`, `ACR_NAME` | |
| `conventional-pr-title.yml` | `types` (one per line) | `feat`, `fix`, `perf`, `deps`, `revert`, `docs`, `refactor`, `test`, `build`, `ci`, `chore`, `style` |

### Setting up a repository

Checklist for every repository that calls these workflows. Apart from the files, everything is configured in the
repository's GitHub **Settings**.

- [ ] **Project files:** the Poe tasks `check` and `docs-build`, `release-please-config.json`,
  `.release-please-manifest.json`, `CHANGELOG.md` and the caller workflows from [Usage](#usage).
- [ ] **General → Pull Requests:** only *Allow squash merging* enabled, with *Default commit message* set to
  **Pull request title and description**. Otherwise single-commit PRs use the commit message instead of the PR title,
  and `BREAKING CHANGE:` / `Release-As:` lines in the description don't reach `main`.
- [ ] **General → Pull Requests:** *Automatically delete head branches* enabled (recommended). This also removes the
  Release PR branch `release-please--branches--main--components--<package-name>` after each release; release-please
  recreates it with the next change on `main`.
- [ ] **Actions → General → Actions permissions:** if actions are restricted, allow `actions/*`,
  `googleapis/release-please-action`, `amannn/action-semantic-pull-request` and `cokeSchlumpf/core--devops-workflows`.
- [ ] **Actions → General → Workflow permissions:** *Allow GitHub Actions to create and approve pull requests*
  enabled. Otherwise release-please can't open the Release PR.
- [ ] **Pages:** *Source* set to **GitHub Actions** (only with `python-docs.yml`). Otherwise the deploy job fails.
- [ ] **Rules → Rulesets:** a branch ruleset for `versions/*` with *Restrict deletions* and *Block force pushes*. Don't
  enable *Restrict updates*, the release workflow pushes to these branches with the `GITHUB_TOKEN`. Don't apply *Block
  force pushes* to all branches: release-please force-pushes its Release PR branch.
- [ ] **Environments** (apps only): an environment per stage (e.g. `uat`) with the variables `AZURE_CLIENT_ID`,
  `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID` and `ACR_NAME` (outputs of `10--sandbox`, no secrets); the repository must
  be listed in `github_app_repositories` of `10--sandbox`, which allows exactly this environment to sign in to Azure.
  Allow `azure/login` and `opentofu/setup-opentofu` if actions are restricted.
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
