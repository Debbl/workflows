# workflows

Reusable GitHub Actions for my pnpm library monorepos - `best-i18n`,
`uni-themes`, and anything started from [`starter-lib`](https://github.com/Debbl/starter-lib).

Before this repo the same 40 lines of pnpm setup were copied into four jobs per
repo; changing the cache key meant editing eight places.

```text
.github/actions/setup              pnpm + Node + store cache + ni + install
.github/workflows/lib-ci.yml       lint & format, typecheck, test matrix
.github/workflows/lib-release.yml  build, pnpm publish, changelogithub
.github/workflows/site-ci.yml      lint & format, typecheck, build
```

## Using them

`.github/workflows/ci.yml` in the consuming repo:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: Debbl/workflows/.github/workflows/lib-ci.yml@main
```

`.github/workflows/release.yml`:

```yaml
name: Release

permissions:
  id-token: write # npm trusted publishing (OIDC)
  contents: write # changelogithub creates the release

on:
  push:
    tags: ['v*']

jobs:
  release:
    uses: Debbl/workflows/.github/workflows/lib-release.yml@main
```

The caller repo has to provide the scripts these call: `lint`, `format:check`,
`build`, `typecheck`, `test`.

## Inputs

### `lib-ci.yml`

| input          | default                                                | |
| -------------- | ------------------------------------------------------ | - |
| `node-version` | `lts/*`                                                | passed to `actions/setup-node` |
| `test-os`      | `'["ubuntu-latest", "windows-latest", "macos-latest"]'` | JSON array the test job fans out over |
| `format-check` | `true`                                                 | run `nr format:check` after the linter |

A repo with no Windows-specific path handling can cut its CI minutes roughly in
three:

```yaml
jobs:
  ci:
    uses: Debbl/workflows/.github/workflows/lib-ci.yml@main
    with:
      test-os: '["ubuntu-latest"]'
```

### `lib-release.yml`

| input          | default | |
| -------------- | ------- | - |
| `node-version` | `lts/*` | passed to `actions/setup-node` |

### `site-ci.yml`

For a statically exported site rather than a package - no test job, no OS
matrix. A static export is the same everywhere, and a docs site has nothing to
unit test. Needs `lint`, `format:check`, `typecheck` and `build`.

| input          | default | |
| -------------- | ------- | - |
| `node-version` | `lts/*` | passed to `actions/setup-node` |
| `format-check` | `true`  | run `nr format:check` after the linter |

## This repo has to stay public

A reusable workflow is only callable from another repository when the repo
holding it is public, or private *and in the same account* with Actions access
granted. `best-i18n/best-i18n` lives under a different owner than `Debbl`, so
public is the only arrangement that works for all the callers. Making this repo
private breaks their CI.

## Pinning

`@main` follows this repo. To pin, tag it and use the tag - callers resolve the
ref at dispatch time, so a tag is the only thing that actually freezes
behaviour.

## License

MIT © [Brendan Dash](https://aiwan.run)
