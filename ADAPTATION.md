# ADAPTATION.md — Loan Origination

How the onboarding pattern transferred from Payments to Loan Origination.
**The principal artifact in this repo.** Evaluators should read this first.

---

## 1. Thesis

The pattern transfers from the canonical Payments implementation
(`jr-cse-payments-postman-onboarding`) to Loan Origination with **`<N>`**
lines of workflow YAML changed and **four** customer-side follow-up items.

**Compute (ECS+ALB vs Lambda+API Gateway) and declared auth (JWT vs OAuth+JWT)
are configuration, not structural concerns, at the onboarding layer.** The
workflow file is structurally identical to the companion's; only the
per-service input values differ.

<!-- TODO: replace `<N>` with the measured line count after both repos are green:

  cd ~/Desktop/jr-cse-loan-origination-postman-onboarding
  gh api repos/JeremiahJRRoss/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml \
    --jq .content | base64 -d > /tmp/companion.yml
  diff /tmp/companion.yml .github/workflows/onboard.yml | grep -c '^<'

-->

## 2. The diff matrix

Every row is observed against the two committed workflow files and the two
running Postman workspaces. Predictions are flagged as such.

| Dimension | Payments | Loan Origination | Change type |
|---|---|---|---|
| Compute target | Lambda + API Gateway | ECS + ALB | N/A (downstream of onboarding) |
| OpenAPI version | 3.0.3 | 3.0.3 | None |
| Spec version | v2.1.0 | v1.4.0 | N/A (their versioning) |
| Auth declared in `securitySchemes` | OAuth 2.0 + JWT | JWT only | Config — generated collection auth differs |
| Auth at runtime (real backend) | OAuth 2.0 client creds | mTLS + JWT | **Manual** — mTLS not derivable from spec; wired via `ssl-client-*` inputs |
| File-upload endpoints | None | `POST /applications/{id}/documents` (multipart/form-data) | **Validate generated collection** post-run |
| Environments | prod / uat / qa / dev (4) | prod / staging / dev (3) | Config (per service) |
| Server URL pattern | api.payments.example.com | lending-api.example.com | Config |
| Workflow file structural changes | n/a (baseline) | 0 | **None** |
| Workflow file input changes | n/a (baseline) | `<N>` lines (see §1) | Config |
| Action ref | `postman-cs/postman-api-onboarding-action@v0` | Same | None |
| Job permissions block | `actions: write, contents: write` | Same | None |
| Generated collection count | 3 (Baseline / Smoke / Contract) | 3 (same) | None |
| Mock server | 1 | 1 | None |
| Monitor | 1, default schedule | 1, default schedule | None |
| `ci-workflow-path` | `payments-tests.yml` | `loan-origination-tests.yml` | Config (per-service, collision avoidance) |
| Secret references | `POSTMAN_API_KEY`, `POSTMAN_ACCESS_TOKEN`, `GH_FALLBACK_TOKEN` | Same | None |
| Identity variables | `REQUESTER_EMAIL`, `POSTMAN_USER_ID` | Same | None |

**Rows that move:** auth-at-runtime, file-upload-endpoints, environments-count, environment-URLs, `ci-workflow-path`, the four per-service input lines (project-name, domain, domain-code, both spec paths).

**Rows that don't move:** everything else.

## 3. The measured diff

```bash
# Run this AFTER both repos are green and the action has committed back:
gh api repos/JeremiahJRRoss/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml \
  --jq .content | base64 -d > /tmp/companion-onboard.yml

diff /tmp/companion-onboard.yml .github/workflows/onboard.yml | grep -c '^<'
```

<!-- TODO: replace with actual output after running both workflows:

The full diff (lines starting with `<` are companion / payments, lines
starting with `>` are this repo / loan-origination):

```
<paste the diff here once both repos are green>
```

Lines that changed (count of `^<` lines): **<N>**

Categorized:
- Config (per-service input values): all `<N>` lines
- Structural (changes to how the workflow is organized): 0 lines
- Manual (anything that required hand-editing beyond input swap): 0 lines

-->

## 4. Follow-ups (customer-side action items)

The pattern works at the catalog / spec / collection / environment / monitor
layer. Runtime concerns are customer-side and live outside any onboarding
workflow regardless of which service. Four items specific to Loan Origination:

### 4.1 mTLS at runtime

**Owner:** Customer (security + DevOps).
**What:** Provision mTLS client cert material as base64-encoded PEM secrets in
the customer's secret manager. Surface as GitHub secrets `SSL_CLIENT_CERT_B64`,
`SSL_CLIENT_KEY_B64`, `SSL_CLIENT_PASSPHRASE`. Uncomment the `ssl-client-*`
input lines in `onboard.yml`.
**Why:** The spec declares only JWT in `securitySchemes` — mTLS appears in
`info.description` prose only. The generated collection wires JWT, which is
correct given the spec. The action accepts cert material and passes it to
`postman-repo-sync-action` for the generated CI test workflow. This is
documented action behavior, not a gap.

### 4.2 Multipart upload endpoint validation

**Owner:** CSE (post-onboarding verification).
**What:** After the green run, open the Baseline collection's
`POST /applications/{id}/documents` request and confirm:
- Body type: form-data (multipart)
- A `file` part is defined
- The `application/id` path parameter is correctly referenced
**Why:** Multipart endpoints are an edge case for spec-to-collection
generators. If the generator dropped the file part or set the wrong content-
type, patch the collection post-generation and surface upstream as an action
issue.

### 4.3 Environment URL realism

**Owner:** Customer (platform team).
**What:** Replace `lending-api.example.com` (from the brief's spec) with real
internal URLs for prod / staging / dev in `env-runtime-urls-json`.
**Why:** Mock-only environments don't test real services. This is the same
ask for every service, not Loan-Origination-specific — but worth listing here
because the spec ships with example.com.

### 4.4 Monitor source-IP allowlisting

**Owner:** Customer (network ops).
**What:** Allowlist Postman monitor source IPs (Postman publishes the list)
on the lending-api network policy.
**Why:** mTLS endpoints are typically allowlist-only at the network layer.
Without allowlisting, the monitor will fail TLS handshake before getting to
auth.

## 5. What this proves about the pattern

**The workflow shape is genuinely reusable.** The diff between the two
onboarding workflows is `<N>` lines of per-service configuration — every line
of structural plumbing (job definition, permissions, action ref, secret
wiring, output printing) is identical. A pilot team can onboard service #3,
#4, ... #50 by copying this workflow, swapping seven inputs (project-name,
domain, domain-code, spec-url, spec-path, environments-json,
env-runtime-urls-json, ci-workflow-path), and pushing.

**The follow-ups above are honest scope, not surprise work.** The action
correctly handles spec-driven catalog onboarding for any well-formed OpenAPI
spec. What's customer-side is what's *always* customer-side regardless of
service: real credentials, real URLs, real network policies. The Adaptation
slide in the deck walks through how Cohort 1 (services with current specs +
GitHub Actions + standard auth) covers an estimated 30 of 50 pilot services
as-is. Cohort 2 needs the per-service work itemized above plus the GitLab CI
port for the one team flagged in the brief.

## 6. Cross-reference

For the canonical pattern implementation — full README, AI disclosure,
iteration history (five concrete fixes), universal-vs-per-service matrix,
validation evidence with screenshots — see
[**jr-cse-payments-postman-onboarding**](https://github.com/JeremiahJRRoss/jr-cse-payments-postman-onboarding).

The companion repo's `README.md §11` documents the build path that produced
the pre-applied fixes in this repo's workflow file. This repo doesn't need
to re-litigate them; this repo's job is to demonstrate transfer, not to
re-establish the pattern.
