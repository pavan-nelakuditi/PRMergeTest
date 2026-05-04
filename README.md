# PRMergeTest

Sandbox consumer repo for testing `postman-cs/postman-api-onboarding-action` with submitted
`postman-cs/postman-bootstrap-action` PRs.

## What this tests

The onboarding composite currently calls:

```yaml
uses: postman-cs/postman-bootstrap-action@main
```

The PR #22 workflow checks out `postman-api-onboarding-action@main`, patches that line on the
runner, and then runs the local patched composite.

The PR #23 workflow uses a branch on the `origin` fork of the orchestrator action:

```yaml
uses: pavan-nelakuditi/postman-api-onboarding-action@codex/bootstrap-pr23-orchestrator-test
```

That branch contains the same orchestrator action with its bootstrap child pinned to bootstrap PR
#23's current head SHA.

- `test-onboarding-bootstrap-pr22.yml` uses bootstrap PR #22 at
  `43d3899c7cbc988af95a207945483562fa3fb427`.
- `test-onboarding-bootstrap-pr23.yml` uses bootstrap PR #23 at
  `13ccbf78f519cdb0d18714f7b339b6f09ff460d7`.

PR #23 is stacked on PR #22, so the PR #23 workflow exercises both changes together.

## How to run

1. Push this folder to a GitHub test repository.
2. Configure repository secrets:
   - `POSTMAN_API_KEY` is required.
   - `POSTMAN_ACCESS_TOKEN` is optional, but enables Bifrost/governance/repo-link behavior.
   - `GH_FALLBACK_TOKEN` is optional. It is mainly needed for workflow-file writes; these tests set
     `generate-ci-workflow: "false"`.
3. Open the Actions tab.
4. Run either manual workflow:
   - `Test onboarding with bootstrap PR 22`
   - `Test onboarding with bootstrap PR 23`

The PR #23 workflow exposes the refresh behavior controls in the manual dispatch UI:

- `collection_sync_mode`: `refresh` keeps tracked collection IDs current; `version` creates or
  reuses release-scoped collections.
- `spec_sync_mode`: `update` keeps one canonical spec current; `version` creates or reuses a
  release-scoped spec.

If `spec_url` is left blank, the workflow uses:

```text
https://raw.githubusercontent.com/<owner>/<repo>/<sha>/openapi/dummy-openapi.yaml
```

That matters for PR #23 because the safe fetch layer requires a public HTTPS `spec-url` and blocks
localhost, private networks, credentials, and non-HTTPS URLs.

If the test repository is private, the default raw GitHub URL may not be fetchable by the bootstrap
action because it does not attach GitHub credentials to `spec-url` requests. In that case, pass a
public HTTPS `spec_url` when manually dispatching the workflow.

The PR #23 workflow uses `repo-write-mode: commit-and-push`, so generated Postman files are
committed and pushed back to the workflow branch. It still uses `generate-ci-workflow: "false"` to
avoid writing a generated CI workflow. Generated `.postman/` and `postman/` output is also uploaded
as a workflow artifact when present.
