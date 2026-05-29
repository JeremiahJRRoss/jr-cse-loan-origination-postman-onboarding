# jr-cse-loan-origination-postman-onboarding

Adaptation analysis for the **Loan Origination** service. The canonical pattern
lives in the companion repo **jr-cse-payments-postman-onboarding**.

**Pattern transfer is the deliverable.** The principal artifact in this repo
is `ADAPTATION.md` — that's what the evaluator should read first.

> **Status:** Workflow ready to run with all five iteration fixes from the
> companion pre-applied. Sections marked `TODO` need observed values once the
> workflow has run in your environment. See `SETUP.md` for the verified
> ~10-minute setup path.

---

## 1. Overview

Onboards the Loan Origination API into Postman using the same orchestrator
action and the same workflow shape as the Payments repo. The diff between the
two onboarding workflows is the pattern-transfer thesis, measured in
`ADAPTATION.md`.

The workflow file is a **structural copy** of the companion's, with only
per-service inputs differing — different compute (ECS+ALB vs Lambda+API
Gateway), different declared auth (JWT vs OAuth+JWT), three environments vs
four, includes a multipart file-upload endpoint. The structural identity
across two services this different is the evidence the pattern scales.

## 2. Why Loan Origination

Maximum contrast with Payments along the two axes that matter for the
brief's pattern-thinking grading criterion:

| Dimension | Payments | Loan Origination |
|---|---|---|
| Compute | Lambda + API Gateway | ECS + ALB |
| Auth declared in spec | OAuth 2.0 + JWT | JWT only |
| Endpoint shapes | Standard JSON | Includes multipart file upload |
| Environments | prod / uat / qa / dev (4) | prod / staging / dev (3) |

If onboarding works for two services this different, the pattern generalizes
— evidence, not assertion. Claims Processing as a third service would have
added another combination but with marginal evidence per slide-minute too low
for the time cost; Claims is referenced in the deck's Adaptation slide as a
third archetype instead.

## 3. What's the Same vs What Changed

The full analysis lives in **ADAPTATION.md**. Two-sentence summary:

The workflow file is structurally identical to the companion's; only
per-service inputs differ. Customer-side follow-ups (mTLS at runtime,
multipart validation, env URL realism, monitor source-IP allowlisting) are
documented as scope, not surprise.

## 4. Customer-Side Requirements (this service)

Four items specific to Loan Origination, beyond the universal requirements
in the companion's §7:

1. **mTLS client cert material in the customer secret manager** — base64-encoded
   PEM certs and keys, surfaced as `SSL_CLIENT_CERT_B64`, `SSL_CLIENT_KEY_B64`,
   `SSL_CLIENT_PASSPHRASE` secrets, then wired by uncommenting the
   `ssl-client-*` lines in `onboard.yml`. The spec declares only JWT in
   `securitySchemes` (mTLS lives in description prose), so this is
   customer-side configuration the action explicitly supports rather than a
   tool gap.
2. **Real environment URLs per env** — the spec ships with
   `lending-api.example.com` URLs from the brief; customer provides real
   internal URLs for prod / staging / dev.
3. **Monitor source-IP allowlisting** — mTLS endpoints are typically
   network-restricted; customer allowlists Postman monitor source IPs.
4. **JWT issuer/audience configuration** — declared in spec but specifics
   (issuer URL, expected audience claim) are customer-side.

## 5. Run Instructions

See `SETUP.md` for the verified setup. Identical workflow trigger pattern as
the companion:

```bash
gh workflow run onboard.yml --repo JeremiahJRRoss/jr-cse-loan-origination-postman-onboarding
gh run watch "$(gh run list --repo JeremiahJRRoss/jr-cse-loan-origination-postman-onboarding --workflow=onboard.yml --limit 1 --json databaseId --jq '.[0].databaseId')"
```

Then validate in the Postman UI per `SETUP.md` Step 7 — six universal checks
plus two loan-specific checks (multipart endpoint, JWT-not-mTLS auth wiring).

## 6. Validation Evidence

**Workspace:** [LND] loan-origination-service ([open in Postman](https://go.postman.co/workspace/6938ecca-fd61-4488-b325-bb022c7e5b83))

**Latest green run:** https://github.com/JeremiahJRRoss/jr-cse-loan-origination-postman-onboarding/actions/runs/26613650633

**Committed artifacts:**
- [`postman/collections/`](postman/collections/) — three git-sync YAML collections (canonical, action-managed: Baseline, Contract, Smoke)
- [`postman/exports/`](postman/exports/) — the same three collections as single-file v2.1 JSON exports (`baseline.json`, `contract.json`, `smoke.json`), provided per the brief
- [`.github/workflows/loan-origination-tests.yml`](.github/workflows/loan-origination-tests.yml) — generated CI workflow

> **⚠️ Known issue (expected, documented):** the generated
> `loan-origination-tests.yml` ships from `postman-repo-sync-action` as a single
> escaped line, so the Actions tab shows **"Invalid workflow file"** for it. This
> is an upstream tooling quirk that repo-sync **regenerates on every run** — so it
> is left in place and documented rather than hand-fixed (any fix or move is
> clobbered by the next onboard run). Same decision as the companion repo. Full
> root-cause and rationale: [§7.C](#7c--loan-specific-observations) below.

**Walkthrough with all five screenshots:** [`docs/VALIDATION-EVIDENCE.md`](docs/VALIDATION-EVIDENCE.md)


## 7. Known Issues and Resolutions

### 7.A — Iteration fixes (pre-applied from companion build)

All five iteration fixes the companion build discovered are pre-applied here:

1. No invalid `variables: write` permission key
2. Both `spec-url` AND `spec-path` provided
3. `gh-fallback-token` wired to a PAT with Workflows write
4. Same PAT verification pattern documented in SETUP
5. PAT's repository access list must include this repo (Step 2 in SETUP)

First green run should be one-shot. If it's not, see SETUP.md `Troubleshooting`.

### 7.B — Annotations on green runs (informational, identical to companion)

Three categories appear on every green run, all benign — same as the companion's
runs because they're produced by the same upstream actions:

- **Node.js 20 runtime deprecation** on `postman-cs/postman-bootstrap-action@v0`
  and `postman-cs/postman-repo-sync-action@v0`. Upstream tooling; forced
  migration to Node 24 in June 2026. Maintainers will publish a Node 24
  version before then.
- **OpenAPI spec linting warnings** on the brief's spec — inline schemas that
  should be `$ref`'d into `components.schemas`. Spec ships unmodified from the
  brief.
- **"Failed to invite requester: Only one role is supported"** — I already
  hold workspace-creator on the new workspace; action tries to add me as
  admin too. Functional impact zero.

### 7.C — Loan-specific observations

The generated CI workflow (`.github/workflows/loan-origination-tests.yml`) ships from `postman-repo-sync-action` as a **single escaped line** (~90 literal `\n`), so GitHub flags it **"Invalid workflow file"** in the Actions tab. Root cause is the same upstream repo-sync escaping documented in the companion's [README §11.A #6](https://github.com/JeremiahJRRoss/jr-cse-payments-postman-onboarding#11a) — same generic generated workflow, same bug. A one-time `\n` → newline decode makes it valid, **but repo-sync regenerates the escaped form on every onboard run** (see §7.D), so a hand-fix — or moving the file out of `.github/workflows/` — is clobbered by the next run. **Decision: leave it as-generated and document it here** (matching the companion); the durable fix is upstream, not an in-repo edit. The file is preserved as honest evidence of the verbatim tool output.

Even when decoded, that workflow's "Resolve Postman Resource IDs" step reads the gitignored `.postman/resources.yaml` and the spec's environments target `example.com`, so it can't go green on a clean checkout — same limitation and same one-line customer fix (un-ignore `.postman/`) as the companion. Mirrors the companion's layer-2 decision; not re-documented here.

Spec-specific observations from the generated collections (verified against
`postman/exports/baseline.json`):

- **Multipart endpoint handled correctly.** The Baseline collection's "Upload a
  document" request (`POST /applications/{applicationId}/documents`) wires
  `Content-Type: multipart/form-data` with a `formdata` body containing the
  `file` part (`type: file`) and the `documentType` part — both the spec's
  `required` form fields. See ADAPTATION.md §4.2.
- **JWT auth wired (not mTLS), as expected.** Auth follows the spec's
  `securitySchemes` (JWT bearer only); mTLS is customer-side config, not in the
  generated collection. See ADAPTATION.md §4.1.

### 7.D — Rerun / idempotency behavior

Identical to the canonical Payments repo (same onboarding action), so the behavior is characterized there rather than re-measured here. The **workspace is reused** (matched via the repo↔workspace git-sync link), but the **spec, collections, mock, and monitor are re-created with new IDs on every run** — no GitHub repo variables are written (`gh variable list` shows only the two hand-set inputs `POSTMAN_USER_ID` / `REQUESTER_EMAIL`; no `POSTMAN_*` resource IDs are stored). Repeated runs therefore **accumulate duplicates** in the one workspace (the R5/R7 sprawl risk); treat onboarding as **create-once per service**. Full evidence and mitigation: companion [`jr-cse-payments-postman-onboarding` README §10](https://github.com/JeremiahJRRoss/jr-cse-payments-postman-onboarding#10).

A loan-specific corollary: repo-sync also regenerates `loan-origination-tests.yml` on every run and re-commits it as the escaped single line, so the item-1 decode does **not** survive a re-run — the durable fix is upstream (repo-sync escaping), not an in-repo edit.

## 8. AI Assistance and Manual Validation

### What AI generated
- Initial draft of the workflow file (built by copying the companion's
  workflow and substituting eight per-service inputs — see ADAPTATION.md for
  the measured diff).
- Initial draft of `ADAPTATION.md` structure.
- Initial draft of this README.

### What I validated manually
- Diffed the two workflow files line by line; confirmed only the expected
  per-service inputs changed.
- Inspected the generated baseline collection's `POST /applications/{applicationId}/documents`
  request to confirm multipart handling.
- Verified mTLS is NOT auto-wired in the generated collection — this is
  consistent with the spec declaring only JWT in `securitySchemes`. mTLS is
  wired via the action's `ssl-client-*` inputs from customer-provided cert
  secrets. Correct behavior, not a gap.

### What AI got wrong (and I corrected)
- All five corrections from the companion's AI disclosure apply identically
  to this repo's workflow file (variables-write, spec-url, gh-fallback-token,
  PAT verification, PAT name-scoping on rename). See the companion's §14 for
  the canonical breakdown. Pre-applying those fixes here was a deliberate
  consolidation step.

### Why this matters
The brief's strongest grading signal is "how little actually changes" in the
second-service analysis. AI-generated parallel authorship would have produced
a deceptively "different" workflow that obscures the structural identity.
The copy-then-diff approach plus the measured 8-input ADAPTATION.md thesis
makes the pattern-transfer claim falsifiable rather than rhetorical.

See the companion's `README.md §14` for the canonical AI disclosure with
specific corrections traceable to `issues-log.md`.

## 9. Recommended future changes (productionizing)

The productionizing roadmap is shared with the canonical repo — the recommendations are **action-level**, so they apply identically here. See [`jr-cse-payments-postman-onboarding`](https://github.com/JeremiahJRRoss/jr-cse-payments-postman-onboarding) README → **"Recommended future changes"**: idempotent re-runs (reuse-or-clean guard), self-healing CI-workflow generation, clean-checkout resource resolution, and dropping the public-URL spec dependency. These are scoped-out of the take-home deliberately — named and scoped, not built.

Loan-specific future items — mTLS client-cert provisioning for CI and monitor source-IP allowlisting for the network-restricted endpoints — are already scoped in [`ADAPTATION.md`](ADAPTATION.md) (§4.1, §4.4); not restated here.

## References

- **Companion repo (canonical pattern):** https://github.com/JeremiahJRRoss/jr-cse-payments-postman-onboarding
- **Brief:** https://github.com/postman-cs/cse-exercise
- **Orchestrator action:** https://github.com/postman-cs/postman-api-onboarding-action
