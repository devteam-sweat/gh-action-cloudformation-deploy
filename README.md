## Deploy CloudFormation Stack Action for GitHub Actions

![Package](https://github.com/guslington/gh-action-cloudformation-deploy/workflows/Package/badge.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Updates a Cloudformation parameter using the previously deployed template

## Usage

```yaml
- name: update cloudformation parameter
  uses: devteam-sweat/gh-action-cloudformation-deploy
  with:
    stack-name: my-stack
    parameter-overrides: |-
      Name=${{ inputs.name }}
      UUID=${{ inputs.uuid }}
```

## Development

Requires Node 20.x (the version CI runs on).

```bash
npm ci
npm test          # vitest with coverage thresholds
npm run all       # build + pack + test — run before pushing
```

| Script | What it does |
|--------|--------------|
| `build` | Compiles `src/` to `lib/` with `tsc` |
| `pack` | Bundles `lib/` into the single-file `dist/index.js` with `ncc` |
| `test` | Runs vitest with v8 coverage (thresholds enforced in [vitest.config.ts](vitest.config.ts)) |
| `all` | `build` + `pack` + `test` |

Source lives in [src/](src/): [index.ts](src/index.ts) is the action entrypoint, [main.ts](src/main.ts) holds the logic, and tests sit in [src/\_\_tests\_\_/](src/__tests__/). Inputs are declared in [action.yml](action.yml) and read via `@actions/core`.

`lib/` is gitignored, but **`dist/` is committed** — it is what GitHub actually runs. You don't need to commit it yourself: the [Package workflow](.github/workflows/package.yml) rebuilds `dist/` on every push to `main` and commits it back as `(chore) updating dist`.

CI on pull requests runs the [Check workflow](.github/workflows/check.yml) (`npm test` and `npm run all`).

## Releasing

1. Merge your PR to `main`.
2. Wait for the Package workflow to push its `(chore) updating dist` commit — tagging before this lands ships a release with a stale bundle.
3. Tag that commit and push:

   ```bash
   git pull
   git tag v0.4.0
   git push origin v0.4.0
   ```

Pushing a `v*` tag triggers the [Release workflow](.github/workflows/release.yml), which creates the GitHub release. Bump `version` in [package.json](package.json) to match if you want it kept in sync.

Consumers should pin a tag (`uses: devteam-sweat/gh-action-cloudformation-deploy@v0.4.0`). There are no floating major-version tags, so a new release requires consumers to bump their reference.