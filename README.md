# PRMergeTest

Sandbox consumer repo for testing submitted `postman-cs/postman-api-onboarding-action` and
`postman-cs/postman-bootstrap-action` PRs.

## What this tests

The onboarding composite currently calls:

```yaml
uses: postman-cs/postman-bootstrap-action@main
```

The PR #22 workflow checks out `postman-api-onboarding-action@main`, patches that line on the
runner, and then runs the local patched composite.

The API onboarding PR #27 workflow uses the submitted orchestrator action directly:

```yaml
uses: postman-cs/postman-api-onboarding-action@5d4df22491a1f529e907733bd0eb243b36b9c2d3
```

That PR fixes the JUnit result step added after repo sync by removing the shell regex guard around the
hardcoded Postman CLI install URL. It calls `postman-cs/postman-bootstrap-action@main`; bootstrap PR
#24 was merged into `main` on May 4, 2026.

- `test-onboarding-bootstrap-pr22.yml` uses bootstrap PR #22 at
  `43d3899c7cbc988af95a207945483562fa3fb427`.
- `test-api-onboarding-pr27.yml` uses API onboarding PR #27 at
  `5d4df22491a1f529e907733bd0eb243b36b9c2d3`.

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
   - `Test API onboarding PR 27`

The API onboarding PR #27 workflow exposes the refresh behavior controls in the manual dispatch UI:

- `collection_sync_mode`: `refresh` keeps tracked collection IDs current; `version` creates or
  reuses release-scoped collections.
- `spec_sync_mode`: `update` keeps one canonical spec current; `version` creates or reuses a
  release-scoped spec.

If `spec_url` is left blank, the workflow uses:

```text
https://raw.githubusercontent.com/<owner>/<repo>/<sha>/openapi/dummy-openapi.yaml
```

That matters because the bootstrap safe fetch layer requires a public HTTPS `spec-url` and blocks
localhost, private networks, credentials, and non-HTTPS URLs.

If the test repository is private, the default raw GitHub URL may not be fetchable by the bootstrap
action because it does not attach GitHub credentials to `spec-url` requests. In that case, pass a
public HTTPS `spec_url` when manually dispatching the workflow.

The API onboarding PR #27 workflow uses `repo-write-mode: commit-and-push`, so generated Postman files are
committed and pushed back to the workflow branch. It still uses `generate-ci-workflow: "false"` to
avoid writing a generated CI workflow. Generated `.postman/` and `postman/` output is also uploaded
as a workflow artifact when present. PR #27 keeps the JUnit XML upload as `postman-test-results` while
using the fixed runner script.
