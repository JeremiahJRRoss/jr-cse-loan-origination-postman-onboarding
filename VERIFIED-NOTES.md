# Verified Action Contract Notes

Captured by inspecting `postman-cs/postman-api-onboarding-action/action.yml` and its
README directly. The source of truth for what *this* repo actually does is its own
`.github/workflows/onboard.yml`; where these notes once conflicted with it, they have
been corrected to match.

## Version pin
- Only `@v0` exists (rolling open-alpha). No immutable `v0.x.y` tags published yet.
- Earlier docs referencing `@v1` were wrong. Use `@v0`.

## Spec input: spec-path vs spec-url
- The action accepts `spec-url` (fetchable HTTPS) and/or `spec-path` (repo-root
  relative, read from the checked-out workspace).
- This scaffold's `onboard.yml` provides **both** `spec-url` and `spec-path` — the
  URL for fetchability, the path as the robust local fallback when the raw URL
  isn't reachable/public before the run.

## Permissions
- Needs `actions: write` and `contents: write` only. There is **no `variables: write`
  permission key** in GitHub Actions — it's a parse error. Repo variables (workspace/
  spec/collection IDs for idempotent re-runs) are persisted via the `github-token` /
  `gh-fallback-token` input (action.yml: "GitHub token used for repo variables and
  generated commits"), not via a workflow permission.

## ci-workflow-path
- Default is `.github/workflows/ci.yml` (NOT `onboard.yml`). The earlier
  "it overwrites itself" claim was wrong. Still set it explicitly per service.

## postman-access-token (critical)
- Session-scoped, manually extracted, expires silently. Without it, workspace
  git-sync (Bifrost), governance assignment, and system-env association are
  SILENTLY SKIPPED. Run looks green; workspace is half-built. #1 failure mode.

## org-mode / workspace-team-id
- If your PMAK's team is org-scoped, the Postman API requires a team ID. Symptom:
  team/org error on first run. Fix: `org-mode: true` + `workspace-team-id`
  (both commented in onboard.yml; set POSTMAN_TEAM_ID variable).

## mTLS (loan-origination relevance)
- The action HAS `ssl-client-cert`, `ssl-client-key`, `ssl-client-passphrase`,
  `ssl-extra-ca-certs` inputs (base64 PEM), passed through to repo-sync for CI
  workflow generation.
- The loan spec declares only JWT in `securitySchemes` (mTLS is in description
  prose), so the generated COLLECTION wires JWT. mTLS is wired via these inputs
  from customer-provided secrets — correct behavior, not a gap.

## starter-assets bug (why we bypass it — decision D2)
- `starter-assets/actions/onboard-service/action.yml` line ~35 passes
  `spec-url: ${{ inputs.spec-path }}` — feeds a local path into the URL input.
  If you pass only a path, it lands in spec-url (expects HTTPS) and fails.
  We call the upstream action directly and use spec-path correctly.

## checkout version
- Official examples use `actions/checkout@v5`. This scaffold uses v5.
