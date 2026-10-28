# Agent Note: Upstream sync credentials

Status: implemented

## Problem

[Run 40628202386](https://github.com/chxcodepro/CloudFlare-ImgBed/actions/runs/40628202386) rejects the upstream addition of `.github/workflows/regression.yml`: the automatic `GITHUB_TOKEN` lacks workflow-file write permission.

## Decision

`.github/workflows/sync-upstream.yml` requires the `SYNC_TOKEN` repository secret. Use a fine-grained personal access token restricted to this repository, with Contents and Workflows read/write permissions. Renew it before expiry.

Checkout disables persisted credentials so the sync action uses `SYNC_TOKEN` when pushing. Compatibility checks and notices continue using `GITHUB_TOKEN` with contents read and issues write permissions. The existing compatibility confirmation gate remains active.

Authenticated sync pushes trigger the existing `deploy-worker.yml` push workflow. The redundant manual dispatch is removed to avoid duplicate deployments. A no-change sync leaves deployment untouched.

## Alternatives considered

- Keep `GITHUB_TOKEN`: avoids managing another credential, but workflow-file updates remain blocked. `actions: write` controls workflow runs, not workflow-file changes.
- Manually sync the fork: clears the current backlog, but later upstream workflow changes recreate the failure.

## Consequences

Full upstream synchronization includes workflow changes and requires an unexpired repository-scoped credential. Missing credentials fail before synchronization. The secret stays in GitHub Actions Secrets; it is never committed.

The change is source-reviewed only, following the user's verification requirement. Operational acceptance requires configuring `SYNC_TOKEN` and completing the Upstream Sync workflow with compatibility confirmation left unchecked unless separately approved.

[GitHub token authentication guidance](https://docs.github.com/en/actions/tutorials/authenticate-with-github_token#granting-additional-permissions)
