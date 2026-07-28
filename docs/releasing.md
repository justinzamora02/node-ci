# Releasing

Releases are manual. The workflow creates an immutable semantic-version tag
and GitHub release before it moves the matching stable major tag.

## Choose a version

Select the next version using semantic versioning:

- Patch (`v1.2.3` to `v1.2.4`) for backward-compatible fixes.
- Minor (`v1.2.3` to `v1.3.0`) for additive, backward-compatible improvements.
- Major (`v1.2.3` to `v2.0.0`) for breaking action-interface changes.

The input must be exactly `vMAJOR.MINOR.PATCH`, with no leading zeroes,
prerelease suffix, or build metadata.

## Before releasing

Merge the intended changes to the default branch and confirm that the
repository's Validate workflow passes. Review the action interface, repository
workflows, documentation, examples, permissions, secret handling,
external-action SHA pins, and backward compatibility. Trigger the release from
the intended default-branch commit.

## Run the workflow

1. Open **Actions**, select **Release**, and choose **Run workflow**.
2. Select the repository's default branch.
3. Enter the version, such as `v1.2.3`, and run it.

The workflow checks out the dispatch commit with full history. Before creating
anything, it validates the version, confirms that the commit is contained in
the current default branch, confirms a clean checkout, and fails a fresh
dispatch if the immutable tag or a conflicting GitHub release already exists.
A retry of the same run can resume only a tag marked as created by that run.

It then performs these operations in order:

1. Create and push immutable tag `v1.2.3`.
2. Create the GitHub release with generated notes.
3. Explicitly create or move stable major tag `v1` to the same commit.

No tag is created or moved until all preflight validation passes.

## Which reference callers use

Most callers should use a stable major reference:

```yaml
uses: justinzamora02/node-ci@v1
```

`v1` advances only through this release workflow and only to compatible v1
releases. Callers needing maximum reproducibility may pin the action's full
commit SHA. Tags such as `v1.2.3` are immutable and must never be deleted,
moved, or recreated.

## Recover a failed major-tag update

If GitHub release creation succeeds but the final major-tag update fails, the
immutable tag and release are valid partial state. Do not delete, move, or
recreate the immutable version tag, and do not start a new dispatch for the
same version; a new run will correctly reject it.

First inspect the release, immutable tag, target commit, failed workflow log,
and any repository tag rules:

```bash
gh release view v1.2.3
git fetch origin --tags
git rev-parse 'v1.2.3^{commit}'
git ls-remote --tags origin refs/tags/v1 refs/tags/v1.2.3
```

After correcting the permission or tag-rule problem, open the original failed
Actions run and choose **Re-run failed jobs**. The workflow verifies that the
immutable tag belongs to the same run and commit, skips the tag and existing
release, and retries the stable-major-tag update. This recovery never rewrites
the immutable tag and keeps every stable-major-tag change inside the release
workflow. Finally, verify that `v1^{commit}` and `v1.2.3^{commit}` resolve to
the same commit.
