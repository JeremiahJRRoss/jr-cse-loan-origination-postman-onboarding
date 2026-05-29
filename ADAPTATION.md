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

< name: Onboard Payments Service to Postman
---
> name: Onboard Loan Origination Service to Postman
3,6c3,5
< # Onboards the Payment Refund API into Postman via the open-alpha orchestrator.
< # This file is the verified working version — every input was validated against
< # the live postman-cs/postman-api-onboarding-action@v0 contract through four
< # debugging iterations. See issues-log.md and README §11 for the corrections.
---
> # Onboards the Loan Origination API into Postman via the open-alpha orchestrator.
> # STRUCTURAL COPY of the companion (jr-cse-payments-postman-onboarding) workflow.
> # Only per-service inputs differ — that's the pattern-transfer thesis, measured.
7a7,9
> # All five iteration fixes from companion are pre-applied. First green run
> # should be one-shot. See ADAPTATION.md for the measured diff vs companion.
> #
9a12
> #              (PAT must include this repo in its access list — see SETUP §2)
11d13
< #               (POSTMAN_TEAM_ID only if your Postman team is org-scoped)
18c20
<       - 'specs/payment-refund-api-openapi.yaml'
---
>       - 'specs/loan-origination-api-openapi.yaml'
32c34
<       - name: Onboard Payments service into Postman
---
>       - name: Onboard Loan Origination service into Postman
36,38c38,48
<           project-name: payment-refund-service
<           domain: payments
<           domain-code: PMT
---
>           # --- per-service deltas vs companion (jr-cse-payments-postman-onboarding) workflow ---
>           project-name: loan-origination-service           # was payment-refund-service
>           domain: lending                                   # was payments
>           domain-code: LND                                  # was PMT
>           spec-url: https://raw.githubusercontent.com/JeremiahJRRoss/jr-cse-loan-origination-postman-onboarding/main/specs/loan-origination-api-openapi.yaml
>           spec-path: specs/loan-origination-api-openapi.yaml
>           environments-json: '["prod","staging","dev"]'    # was prod/uat/qa/dev (3 envs, not 4)
>           env-runtime-urls-json: '{"prod":"https://lending-api.example.com/v1","staging":"https://lending-api-staging.example.com/v1","dev":"https://lending-api-dev.example.com/v1"}'
>           ci-workflow-path: .github/workflows/loan-origination-tests.yml
>           # --- everything below is identical to companion ---
> 
41,53d50
< 
<           # BOTH spec-url AND spec-path are required. The action.yml declares both
<           # as optional ("provide either"), but the underlying bootstrap action
<           # requires spec-url at runtime. spec-path is used for repo-side metadata.
<           spec-url: https://raw.githubusercontent.com/JeremiahJRRoss/jr-cse-payments-postman-onboarding/main/specs/payment-refund-api-openapi.yaml
<           spec-path: specs/payment-refund-api-openapi.yaml
< 
<           # 4 environments from the spec's servers block
<           environments-json: '["prod","uat","qa","dev"]'
<           env-runtime-urls-json: '{"prod":"https://api.payments.example.com/v2","uat":"https://api-uat.payments.example.com/v2","qa":"https://api-qa.payments.example.com/v2","dev":"https://api-dev.payments.example.com/v2"}'
< 
<           # Generated CI test workflow — runs the collections on a schedule.
<           # Explicit path avoids any future default change (current default: ci.yml).
55d51
<           ci-workflow-path: .github/workflows/payments-tests.yml
57,59c53,54
<           # --- ORG-MODE TEAMS ONLY ---
<           # Uncomment if your first run errors on team/org. Sets POSTMAN_TEAM_ID
<           # as a repo variable, then uncomment these two lines:
---
>           # --- ORG-MODE TEAMS ONLY (same as companion) ---
>           # Uncomment if first run errors on team/org. Set POSTMAN_TEAM_ID variable first.
62a58,69
>           # --- mTLS (customer-side cert provisioning, OPTIONAL) ---
>           # The spec declares only JWT in securitySchemes (mTLS lives in
>           # info.description prose). The generated COLLECTION wires JWT, which
>           # is correct behavior given the spec. The action DOES accept client
>           # cert material as base64 PEM via ssl-client-* inputs and passes it
>           # to repo-sync for the generated CI test workflow. To wire mTLS at
>           # runtime, customer provisions cert material as secrets and
>           # uncomments the lines below:
>           # ssl-client-cert: ${{ secrets.SSL_CLIENT_CERT_B64 }}
>           # ssl-client-key: ${{ secrets.SSL_CLIENT_KEY_B64 }}
>           # ssl-client-passphrase: ${{ secrets.SSL_CLIENT_PASSPHRASE }}
> 
67,71c74,76
<           # REQUIRED: gh-fallback-token is a fine-grained PAT with Contents +
<           # Workflows + Actions + Variables write scopes. The default GITHUB_TOKEN
<           # cannot write to .github/workflows/ by design (supply-chain protection);
<           # this PAT lets the action materialize the generated CI test workflow.
<           # See SETUP.md Step 2 for PAT creation instructions.
---
>           # REQUIRED: see companion's onboard.yml for the rationale.
>           # Default GITHUB_TOKEN cannot write to .github/workflows/; this PAT
>           # gives the action the ability to materialize the generated CI workflow.


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
