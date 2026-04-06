# PR Issue Check Action

`pr-check` is a lightweight composite GitHub Action that enforces a simple rule for pull requests:
**a PR should be linked to at least one issue**.

The action is designed for public repository use and keeps behavior explicit and predictable.

## What it does

- Runs during PR workflows.
- Ignores non-PR events.
- Skips draft PRs.
- Skips PRs whose source branch matches configured exclusion patterns.
- Skips PRs that already have configured labels.
- Checks the pull request `closingIssuesReferences` relation via GraphQL.
- If no linked issue is found:
  - posts or updates a reminder comment, and
  - fails the action step.
- If a linked issue exists:
  - deletes the previous reminder comment from this action (if present), and
  - succeeds.

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `github-token` | **true** |  | GitHub token used to call the REST and GraphQL APIs. |
| `exclude-branches` | false | `dependabot/**` | Comma or newline separated branch patterns to skip. |
| `exclude-labels` | false | `skip-issue-check,documentation` | Comma separated labels that skip the check. |
| `comment-marker` | false | `<!-- pr-check:missing-linked-issue -->` | Hidden marker used to locate and update/remove the reminder comment. |
| `missing-issue-message` | false | A generic professional reminder message (see `action.yml`). | Content shown when a linked issue is missing. |

## Recommended workflow permissions

Set permissions in your workflow file to limit scope:

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: write
```

- `issues: write` is required to post/update/delete the PR comment.

## Recommended trigger

Use `pull_request_target` if you want the action to comment on forked pull requests:

```yaml
on:
  pull_request_target:
    types: [opened, edited, synchronize, reopened, ready_for_review, labeled, unlabeled]
```

> [!WARNING]
> `pull_request_target` runs with base-repository privileges. Keep this action in an isolated job that does **not** check out or execute untrusted PR code. If you do not need fork-comment support, `pull_request` is the safer default.

If you rely on `exclude-labels`, include `labeled` and `unlabeled` so the check re-runs when labels change.

## Usage example

```yaml
name: PR checks

on:
  pull_request_target:
    types: [opened, edited, synchronize, reopened, ready_for_review, labeled, unlabeled]

permissions:
  contents: read
  pull-requests: read
  issues: write

jobs:
  check-pr-link:
    runs-on: ubuntu-latest
    steps:
      - name: Require linked issue
        uses: svtter/pr-check@<full-commit-sha>
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          exclude-branches: dependabot/**, renovate/**
          exclude-labels: skip-issue-check, documentation
```

For production use, pin this action to a full commit SHA. If you prefer the convenience of a moving major version, you can also use `svtter/pr-check@v1`.

## Behavior notes

- `exclude-branches` and `exclude-labels` are optional and support multiple values.
- The action only manages reminder comments authored by `github-actions[bot]` whose body starts with the configured marker.
- The action fails only when no linked issue is found and a comment is posted/updated.

## License

This repository is released under the Apache-2.0 License. See `LICENSE`.
