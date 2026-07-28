# Contributing

## Prerequisites

Use Git, `yamllint`, [action-validator](https://github.com/mpalmer/action-validator),
[actionlint](https://github.com/rhysd/actionlint), Bash, and ShellCheck. GitHub
CLI is useful when checking releases or action tags.

## Propose a change

Open an issue or pull request that describes the caller need, the smallest
generally useful interface, and any security or compatibility impact. Avoid new
inputs until a real caller needs the variation, and keep project-specific
build, Docker, publishing, deployment, and application-release behavior out of
the action.

Implement the smallest change that satisfies the use case. Prefer native GitHub
Actions features and official actions. Update the relevant documentation and
all caller examples whenever an input, output, permission, secret, default, or
behavior changes.

## Validate locally

From the repository root, run:

```bash
yamllint_config=$'extends: default\nrules:\n  line-length: disable'
yamllint --strict --config-data "$yamllint_config" action.yml .github examples
action-validator action.yml
actionlint .github/workflows/*.yml examples/*.yml
```

Run `bash -n` and ShellCheck against every changed multiline `run` script. The
CI validation workflow checks the action metadata, YAML, and workflow syntax,
but local checks make review faster.

## Security and compatibility

- Require callers to grant only `contents: read`, and give repository workflows
  and jobs only the permissions they need.
- Do not declare unused secrets or use `secrets: inherit`.
- Never print, persist, or cache secrets, credentials, token-bearing
  configuration, or `node_modules`.
- Never execute pull-request code through `pull_request_target`.
- Preserve existing action behavior within a stable major release. Breaking
  interface changes require a new major; additive compatible changes remain in
  the current major.

Pin every external action to a full 40-character commit SHA with a release
comment. Resolve the tag from the action's official repository, confirm that
the commit exists there, and review the official release and signature. For
example:

```bash
git ls-remote https://github.com/actions/checkout.git \
  refs/tags/v6.1.0 'refs/tags/v6.1.0^{}'
```

Do not copy an unverified SHA from an unrelated workflow.

## Pull requests and releases

Pull requests should explain the caller need, compatibility and permission
effects, documentation changes, and validation performed. Keep unrelated
changes out of the pull request.

Maintainers release accepted changes using the
[release process](docs/releasing.md). Immutable version tags never move, and
stable major tags move only as part of that documented process.
