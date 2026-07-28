# Node CI action

`action.yml` checks out the caller repository, installs locked dependencies,
and runs the caller-owned `<package-manager> run ci` command. It does not
prescribe lint, test, typecheck, build, artifact, Docker, publishing, or
deployment behavior.

## Caller contract

Run the action on `ubuntu-latest` with `contents: read` permission and a
20-minute job timeout. The project directory must contain a valid
`package.json` with a top-level `packageManager` string naming a supported
manager and an exact semantic version:

```json
{
  "packageManager": "pnpm@10.14.0",
  "scripts": {
    "ci": "pnpm run lint && pnpm run test && pnpm run build"
  }
}
```

Ranges and moving labels such as `pnpm@^10`, `yarn@latest`, or `npm@11` are
rejected. Exact prerelease or build versions are accepted when they follow
semantic-version syntax.

By default, the repository root must also contain `.nvmrc`. It is passed to
`actions/setup-node`; use an exact Node.js version when reproducibility matters:

```text
22.17.1
```

The supported package managers and required lockfiles are:

| `packageManager` | Required lockfile | Install command |
| --- | --- | --- |
| `npm@...` | `package-lock.json` | `npm ci` |
| `pnpm@...` | `pnpm-lock.yaml` | `pnpm install --frozen-lockfile` |
| `yarn@...` | `yarn.lock` | `yarn install --immutable` |

Corepack activates the declared pnpm or Yarn version. The Node distribution's
npm command is used for npm projects.

## The `ci` script

The action deliberately has one repository-owned entry point. Compose the
operations that apply to the project in its `ci` script:

```json
{
  "scripts": {
    "ci": "npm run lint && npm run typecheck && npm test"
  }
}
```

For a package that only needs tests, `"ci": "yarn test"` is sufficient. The
action neither requires nor infers individual scripts.

## Inputs

Only two optional inputs are accepted:

| Input | Default | Purpose |
| --- | --- | --- |
| `node-version-file` | `.nvmrc` | Node.js version file, relative to repository root |
| `working-directory` | `.` | Directory containing `package.json` and its lockfile |

### Root project

The action performs checkout, so the caller does not need a separate checkout
step:

```yaml
permissions:
  contents: read

jobs:
  node-ci:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Run Node CI
        uses: justinzamora02/node-ci@v1
```

### Monorepo package

Paths are relative to the checked-out repository:

```yaml
permissions:
  contents: read

jobs:
  web-ci:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Run web CI
        uses: justinzamora02/node-ci@v1
        with:
          node-version-file: apps/web/.nvmrc
          working-directory: apps/web
```

The selected package must contain its own `package.json`, lockfile, and `ci`
script.

## Caching

`actions/setup-node` caches the package manager's download cache or content
store, keyed using the selected lockfile. It does not cache `node_modules`.
Restoring `node_modules` can preserve install-time side effects, platform-bound
files, stale dependency trees, and undeclared changes. Re-running the package
manager's locked install is a simpler and safer reproducibility boundary.

## Permissions and security

The action defines no secret inputs and does not implicitly inherit secrets.
The caller grants only `contents: read`; checkout disables persisted Git
credentials. External actions are pinned to verified full commit SHAs.
Pull-request code runs under `pull_request`, not `pull_request_target`. The
package-manager value is validated before it is used, and no token,
configuration containing credentials, or `node_modules` directory is cached.

Composite actions cannot declare a runner, job timeout, or token permissions,
so the caller owns the documented `ubuntu-latest`, `timeout-minutes: 20`, and
`contents: read` settings.

## Versioning and upgrades

Callers normally use `@v1`. Additive, backward-compatible improvements remain
within that major version; breaking changes to inputs or behavior require a new
major. Upgrade between major tags deliberately after reviewing the release
notes. For maximum reproducibility, replace `@v1` with an immutable commit SHA.
Immutable release tags such as `v1.2.3` never move, while stable major tags move
only through the release workflow.

## Common failures

- `Missing package.json`: set `working-directory` to the directory containing
  `package.json`, or add the missing file.
- `Invalid packageManager`: add an exact value such as `npm@11.4.2`; do not use
  a range or moving label.
- `Unsupported package manager`: use npm, pnpm, or Yarn.
- `Missing lockfile`: generate the named lockfile with the declared package
  manager and commit it.
- A setup-node error about the version file: add `.nvmrc` at the repository
  root or pass the correct `node-version-file`.
- A package-manager error about a missing `ci` script: define `scripts.ci` in
  the selected `package.json`.
- A frozen or immutable install error: regenerate and commit the lockfile so it
  matches `package.json`; do not loosen the install command.

## Complete caller

```yaml
---
name: Node CI

'on':
  pull_request:
  push:
    branches:
      - master

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  node-ci:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Run Node CI
        uses: justinzamora02/node-ci@v1
        with:
          node-version-file: apps/web/.nvmrc
          working-directory: apps/web
```

Adjust or remove the two overrides as needed.
