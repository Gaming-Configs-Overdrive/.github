# Gaming-Configs-Overdrive/.github

Org-wide community files, the org profile (`profile/README.md`), shared
[reusable workflows](https://docs.github.com/actions/using-workflows/reusing-workflows)
for every game repo and a shared Renovate preset. Contract:
[`game-config-skeleton/docs/ARCHITECTURE.md`](https://github.com/Gaming-Configs-Overdrive/game-config-skeleton/blob/main/docs/ARCHITECTURE.md).

> **Versioning:** callers pin `@v1`. After merging to `main`, set the tag `v1`
> and move it forward (`git tag -f v1 && git push -f origin v1`) after every
> compatible change. Breaking interface changes get `v2`.

## Workflows

| Workflow | Inputs | Does |
|---|---|---|
| `game-lint.yml` | – | `npm run format:check`, `npm run validate`, `npm test`, yamllint (without `node_modules`, `.github`), pyinilint (`*.ini`), JSON syntax (without `node_modules`, `package-lock.json`). File-based jobs skip with a notice if nothing matches. |
| `game-release.yml` | `installer-ref` (string, `v1`), `include-installer` (bool, `true`) | Version = first `## [x.y.z]` in `CHANGELOG.md`. Skips if tag `vX.Y.Z` exists. Fails on missing version or empty notes. `npm ci && npm run build`, adds `gco-install.exe`, `gco-install`, `install.bat`, `install.sh` from `game-config-installer@installer-ref` to the root of `dist/fullrelease.zip`, regenerates `dist/SHA256SUMS`, attests provenance, creates the release. Never runs in `game-config-skeleton`. |
| `game-pages.yml` | – | Jekyll build (README as index) and GitHub Pages deploy. |
| `game-template-sync.yml` | secret `TEMPLATE_SYNC_TOKEN` (optional) | Opens a PR `🔧 chore(template): sync skeleton` with changes from `game-config-skeleton@main`. Never runs in the skeleton. |

All actions are pinned by commit SHA; Node comes from the caller's `.nvmrc`.
`lint.yml` runs `actionlint` on this repo.

## Callers (game repo)

```yaml
# .github/workflows/lint.yml
name: Lint
on:
  pull_request:
  push:
    branches: [main]
permissions:
  contents: read
jobs:
  lint:
    uses: Gaming-Configs-Overdrive/.github/.github/workflows/game-lint.yml@v1
```

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
permissions:
  contents: write
  id-token: write
  attestations: write
jobs:
  release:
    uses: Gaming-Configs-Overdrive/.github/.github/workflows/game-release.yml@v1
    with:
      installer-ref: v1
      include-installer: true
```

```yaml
# .github/workflows/pages.yml
name: Pages
on:
  push:
    branches: [main]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  pages:
    uses: Gaming-Configs-Overdrive/.github/.github/workflows/game-pages.yml@v1
```

```yaml
# .github/workflows/template-sync.yml
name: Template sync
on:
  schedule:
    - cron: "0 4 * * 1"
  workflow_dispatch:
permissions:
  contents: write
  pull-requests: write
jobs:
  sync:
    uses: Gaming-Configs-Overdrive/.github/.github/workflows/game-template-sync.yml@v1
    secrets: inherit
```

`TEMPLATE_SYNC_TOKEN`: PAT or GitHub App token with `workflow` scope. Without it
`github.token` is used, which cannot push changes to `.github/workflows/`.

## Renovate

```json
{ "extends": ["github>Gaming-Configs-Overdrive/.github:renovate-config"] }
```

Weekly, non-major updates grouped, label `dependencies`, commits
`⬆️ chore(deps): …`, GitHub Actions pinned to digests.
