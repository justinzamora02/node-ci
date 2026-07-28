# node-ci

A small, secure GitHub Action that installs locked Node.js dependencies and
runs a repository-owned `ci` package script. It supports npm, pnpm, and Yarn
without prescribing project-specific lint, test, typecheck, or build commands.

## Usage

The caller needs a `.nvmrc`, a supported lockfile, and an exact
`packageManager` value in `package.json`.

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

See the [Node CI guide](docs/node-ci.md) and
[complete caller example](examples/node-ci-caller.yml).

## Versioning

Callers normally use a stable major tag such as `@v1`. Breaking action-interface
changes require a new major version; additive, backward-compatible improvements
stay in the current major. Version tags such as `v1.2.3` are immutable. Stable
major tags move only through the repository's release workflow. Callers that
need maximum reproducibility may pin an immutable commit SHA.

See [CONTRIBUTING.md](CONTRIBUTING.md) for changes and
[the release guide](docs/releasing.md) for releases.

## Roadmap

Docker-oriented automation may be published separately later. It is not
implemented here; this repository remains focused on Node.js CI.
