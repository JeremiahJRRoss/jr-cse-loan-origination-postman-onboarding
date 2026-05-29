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
gh workflow run onboard.yml --repo <OWNER>/jr-cse-loan-origination-postman-onboarding
gh run watch "$(gh run list --repo <OWNER>/jr-cse-loan-origination-postman-onboarding --workflow=onboard.yml --limit 1 --json databaseId --jq '.[0].databaseId')"
```

Then validate in the Postman UI per `SETUP.md` Step 7 — six universal checks
plus two loan-specific checks (multipart endpoint, JWT-not-mTLS auth wiring).

## 6. Validation Evidence

**Workspace:** [LND] loan-origination-service ([open in Postman](https://go.postman.co/workspace/6938ecca-fd61-4488-b325-bb022c7e5b83))

**Latest green run:** https://github.com/JeremiahJRRoss/jr-cse-loan-origination-postman-onboarding/actions/runs/26613650633

**Committed artifacts:**
- [`postman/collections/`](postman/collections/) — three JSON exports
- [`.github/workflows/loan-origination-tests.yml`](.github/workflows/loan-origination-tests.yml) — generated CI workflow

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

The generated CI workflow (`.github/workflows/loan-origination-tests.yml`) shipped as a single escaped line and needed the same `\n` → newline decode as the companion — see the companion's [README §11.A #6](https://github.com/JeremiahJRRoss/jr-cse-payments-postman-onboarding#11a) for the root cause (it's the same generic generated workflow, so the same upstream bug).

Even now decoded, that workflow's "Resolve Postman Resource IDs" step reads the gitignored `.postman/resources.yaml` and the spec's environments target `example.com`, so it can't go green on a clean checkout — same limitation and same one-line customer fix (un-ignore `.postman/`) as the companion. Mirrors the companion's layer-2 decision; not re-documented here.

<!-- TODO: after running the workflow and validating the UI, add any observations specific to this spec:

- Whether the generated baseline collection correctly handles the
  `POST /applications/{id}/documents` multipart endpoint (file part defined,
  content-type set correctly)
- Whether JWT auth was wired correctly in the generated collections (expected:
  yes — securitySchemes declares JWT bearer)
- Anything else that emerged from the six UI checks plus the two
  loan-specific checks in SETUP §7

-->

## 8. AI Assistance and Manual Validation

### What AI generated
- Initial draft of the workflow file (built by copying the companion's
  workflow and substituting six per-service inputs — see ADAPTATION.md for
  the measured diff).
- Initial draft of `ADAPTATION.md` structure.
- Initial draft of this README.

### What I validated manually
- Diffed the two workflow files line by line; confirmed only the expected
  per-service inputs changed.
- Inspected the generated baseline collection's `POST /applications/{id}/documents`
  request to confirm multipart handling.
- Verified mTLS is NOT auto-wired in the generated collection — this is
  consistent with the spec declaring only JWT in `securitySchemes`. mTLS is
  wired via the action's `ssl-client-*` inputs from customer-provided cert
  secrets. Correct behavior, not a gap.

### What AI got wrong (and I corrected)
- **AI initially suggested re-authoring the workflow from scratch** rather
  than copying the companion's and diffing the inputs. Corrected: the
  pattern-transfer story is undermined by parallel authorship. Copy-then-diff
  makes "how little changed" measurable. This was a deliberate process
  decision, not a bug, but worth surfacing as an example of why I held the
  line on the copy approach.
- All five corrections from the companion's AI disclosure apply identically
  to this repo's workflow file (variables-write, spec-url, gh-fallback-token,
  PAT verification, PAT name-scoping on rename). See the companion's §14 for
  the canonical breakdown. Pre-applying those fixes here was a deliberate
  consolidation step.

### Why this matters
The brief's strongest grading signal is "how little actually changes" in the
second-service analysis. AI-generated parallel authorship would have produced
a deceptively "different" workflow that obscures the structural identity.
The copy-then-diff approach plus the measured `<N>`-line ADAPTATION.md thesis
makes the pattern-transfer claim falsifiable rather than rhetorical.

See the companion's `README.md §14` for the canonical AI disclosure with
specific corrections traceable to `issues-log.md`.

## References

- **Companion repo (canonical pattern):** https://github.com/JeremiahJRRoss/jr-cse-payments-postman-onboarding
- **Brief:** https://github.com/postman-cs/cse-exercise
- **Orchestrator action:** https://github.com/postman-cs/postman-api-onboarding-action
