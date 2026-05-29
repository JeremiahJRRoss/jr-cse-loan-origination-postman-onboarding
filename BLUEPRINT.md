# BLUEPRINT — jr-cse-loan-origination-postman-onboarding

> **Repo role:** Adaptation analysis per the Postman CSE take-home brief.
> Demonstrates that the onboarding pattern from `jr-cse-payments-postman-onboarding`
> transfers to a service with meaningfully different characteristics
> (ECS+ALB compute, mTLS+JWT auth) with minimal change.
>
> **Companion:** `jr-cse-payments-postman-onboarding` is the canonical pattern source.
> This repo's job is to document deltas, not re-establish the pattern.
>
> **How to use this file:**
> - Each PR has detailed instructions AND a copy-paste **execute prompt**.
> - Open this repo in Claude Code (`claude`), copy the execute prompt for the
>   target PR, fill placeholders, paste.
> - Claude Code auto-loads `CLAUDE.md` and reads `BLUEPRINT.md` when prompted.

---

## 0. Source of truth + companion

The strategic plan lives in the candidate's claude.ai project workspace. The
canonical pattern lives in the companion repo. This BLUEPRINT focuses on **what
changes**, not on re-explaining the pattern.

The brief explicitly grades this:

> "The strongest signal is when your second-spec analysis demonstrates how
> little actually changes."

---

## 1. Critical constraints (same as companion)

1. **Mixed infrastructure** — this service: ECS+ALB. Contrast with Payments' Lambda+API Gateway.
2. **Mixed CI/CD** — GitHub Actions here; GitLab CI team is the third group.
3. **Inconsistent OpenAPI coverage** — this spec exists and is current.
4. **Platform team stretched thin** — same constraint.
5. **Governance, not utilization** — same constraint.

---

## 2. Decisions affecting this repo

| # | Decision | Why it matters here |
|---|---|---|
| D1 | Loan Origination is the contrast service | ECS+ALB compute, mTLS+JWT auth — max contrast with Payments |
| D2 | Use `postman-cs/postman-api-onboarding-action@v0` directly | Same orchestrator |
| D3 | One repo per service | This BLUEPRINT covers one repo |

D1 rationale: if onboarding works for two services with different compute AND
different auth, you have evidence (not assertion) that it scales. A third
combination (Claims) had marginal evidence-per-slide-minute too low for the
time cost; Claims is referenced in the deck's Adaptation slide instead.

---

## 3. Pre-PR setup (manual, before any Claude Code work)

| # | Step | Verify |
|---|---|---|
| S1 | Companion repo `jr-cse-payments-postman-onboarding` exists with at least PR 1 merged | You can link the green run |
| S2 | Same `POSTMAN_API_KEY` and `POSTMAN_ACCESS_TOKEN` obtained (one set, reusable) | `gh secret list --repo <companion>` shows both |
| S3 | This repo created on GitHub, cloned locally | `git remote -v` shows the right origin |
| S4 | Both secrets set on THIS repo: `gh secret set POSTMAN_API_KEY` + `gh secret set POSTMAN_ACCESS_TOKEN` | `gh secret list` shows both |

Pro tip: set secrets at org level once instead of twice:

```bash
gh secret set POSTMAN_API_KEY --org <org> --visibility selected \
  --repos jr-cse-payments-postman-onboarding,jr-cse-loan-origination-postman-onboarding
gh secret set POSTMAN_ACCESS_TOKEN --org <org> --visibility selected \
  --repos jr-cse-payments-postman-onboarding,jr-cse-loan-origination-postman-onboarding
```

---

## 4. PR sequence

Four PRs. Intentionally smaller than companion because this repo's job is to
demonstrate transfer, not re-establish the pattern.

Each PR has: Goal, Prerequisites, Detailed steps, Acceptance gate, Commit
message, and Execute prompt.

---

### PR 0 — Scaffolding (with adaptation framing)

**Goal:** Lay down static files, with adaptation framing baked in. After this
PR, `tree` matches the layout below; `CLAUDE.md` cross-references the companion repo.

**Prerequisites:** S1-S4 complete. Empty repo cloned locally. `CLAUDE.md` and
`BLUEPRINT.md` already committed.

**Target file tree:**

```
.
├── CLAUDE.md                      ← already exists; opens with companion cross-ref
├── BLUEPRINT.md                   ← already exists
├── README.md                      ← stub: "adaptation analysis; see companion for pattern"
├── .gitignore
├── .claude/
│   ├── settings.json
│   └── commands/
│       ├── log-issue.md
│       ├── run-and-watch.md
│       ├── validate-postman.md
│       └── compare-to-payments.md   ← NEW: helps build diff matrix
├── .github/
│   └── workflows/
│       └── .gitkeep               ← workflow added in PR 1
├── specs/
│   └── loan-origination-api-openapi.yaml   ← from brief, unmodified
├── postman/
│   └── collections/
│       └── .gitkeep               ← repo-sync populates in PR 1
├── service.config.yml             ← pre-notes mTLS + ECS deltas
└── issues-log.md
```

**Detailed steps:**

1. Create the file tree above.
2. **Spec file** — fetch from brief:
   ```bash
   mkdir -p specs
   curl -fsSL -o specs/loan-origination-api-openapi.yaml \
     https://raw.githubusercontent.com/postman-cs/cse-exercise/main/specs/loan-origination-api-openapi.yaml
   ```
   Verify non-empty and starts with `openapi: 3.0.3`.
3. **`.gitignore`** — same as companion (Node + OS + env patterns).
4. **`.claude/settings.json`** — same as companion's (see companion BLUEPRINT §5).
5. **Slash commands** — copy the three from companion (`log-issue`,
   `run-and-watch`, `validate-postman`); add the new `/compare-to-payments`
   (see §6 below).
6. **`service.config.yml`** — see §7 below.
7. **`issues-log.md`** — same format as companion.
8. **`README.md`** stub:
   ```markdown
   # jr-cse-loan-origination-postman-onboarding

   Adaptation analysis for the Loan Origination service. See companion repo
   `jr-cse-payments-postman-onboarding` for the canonical pattern implementation.

   Pattern transfer is the deliverable. See ADAPTATION.md (created in PR 2)
   for the diff matrix and follow-up items.

   Detailed README populated in PR 3.
   ```
9. `tree -a -I '.git'` and confirm.
10. Commit and push.

**Acceptance gate:**

- [ ] `tree -a -I '.git'` matches the target layout
- [ ] Spec file verbatim from brief
- [ ] `CLAUDE.md` cross-references the companion repo URL (not a placeholder)
- [ ] No secrets in any committed file

**Commit message:**

```
PR 0: Scaffold loan-origination adaptation repo

- Drop in loan-origination OpenAPI spec from brief
- service.config.yml pre-notes mTLS + ECS deltas
- /compare-to-payments slash command for diff matrix work
- README stub frames repo as adaptation, not duplicate pattern

Refs: BLUEPRINT.md §4 PR 0
```

#### Execute prompt for PR 0

```
Execute PR 0 (Scaffolding) from BLUEPRINT.md.

FILL IN BEFORE PASTING:
- COMPANION_REPO_URL: <https://github.com/<owner>/jr-cse-payments-postman-onboarding>

Context:
- Empty repo. CLAUDE.md and BLUEPRINT.md are already committed.
- Manual setup S1-S4 complete; secrets confirmed via `gh secret list`.
- Companion repo `jr-cse-payments-postman-onboarding` exists with at least PR 1 merged.

Steps (per BLUEPRINT.md §4 PR 0):
1. Fetch the spec from the brief:
   curl -fsSL -o specs/loan-origination-api-openapi.yaml \
     https://raw.githubusercontent.com/postman-cs/cse-exercise/main/specs/loan-origination-api-openapi.yaml
   Verify non-empty and starts with `openapi: 3.0.3`.
2. Create the rest of the file tree.
3. Source contents from:
   - .claude/settings.json → same as companion's BLUEPRINT §5
   - .claude/commands/log-issue.md, run-and-watch.md, validate-postman.md → same as companion's BLUEPRINT §6
   - .claude/commands/compare-to-payments.md → BLUEPRINT.md §6
   - service.config.yml → BLUEPRINT.md §7
   - issues-log.md → same format as companion's BLUEPRINT §8
   - README.md stub → BLUEPRINT.md §4 PR 0 step 8
   - .gitignore → same as companion's
4. Update CLAUDE.md: replace `<COMPANION_REPO_URL>` placeholder at top with the actual URL.
5. Before committing: `tree -a -I '.git'` and show me.
6. After my approval: commit per BLUEPRINT.md §4 PR 0 message template, push.
7. Confirm acceptance gate (4 boxes).

Do NOT push without showing me the file tree first.
```

---

### PR 1 — Workflow file + first green run (structural copy with deltas)

**Goal:** Working `.github/workflows/onboard.yml` that is a **structural copy of
the companion's**, with only the per-service inputs differing. The diff should
fit on one slide.

**Prerequisites:** PR 0 merged. Both secrets set on this repo. Companion's
PR 1 workflow is green.

**Detailed steps:**

1. `git checkout -b pr-1-workflow`.
2. Fetch companion's workflow:
   ```bash
   gh api repos/<OWNER>/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml \
     --jq .content | base64 -d > /tmp/companion-onboard.yml
   ```
3. Copy to this repo:
   ```bash
   mkdir -p .github/workflows
   cp /tmp/companion-onboard.yml .github/workflows/onboard.yml
   ```
4. Change ONLY these inputs:
   - `project-name`: `loan-origination-service`
   - `domain`: `lending`
   - `domain-code`: `LND`
   - `spec-url`: point at this repo's spec raw URL
   - `environments-json`: `'["prod","staging","dev"]'`
   - `env-runtime-urls-json`:
     ```yaml
     env-runtime-urls-json: |
       {
         "prod":    "https://lending-api.example.com/v1",
         "staging": "https://lending-api-staging.example.com/v1",
         "dev":     "https://lending-api-dev.example.com/v1"
       }
     ```
   - `ci-workflow-path`: `.github/workflows/loan-origination-tests.yml`
5. Diff the two files:
   ```bash
   diff /tmp/companion-onboard.yml .github/workflows/onboard.yml
   ```
   Confirm only ~6 input lines changed (plus surrounding context). Log line
   count in `issues-log.md`.
6. Commit and push.
7. `/run-and-watch`.
8. If failed: diagnose, log, iterate.

**Expected friction points** (anticipated; verify in practice and log either way):

- **mTLS auth at runtime.** Spec's `securitySchemes` declares JWT bearer only;
  mTLS is mentioned in `info.description`. Generated collection wires JWT but
  not cert material. Known gap; log it as a follow-up regardless of whether
  the run fails.
- **File-upload endpoint** (`POST /applications/{id}/documents` uses
  `multipart/form-data`). Verify the generated baseline collection handles this
  correctly by opening the request in Postman UI.

**Acceptance gate:**

- [ ] Latest run on `main` is green
- [ ] Workflow diff vs companion is only the 6 expected input lines (verified with `diff`)
- [ ] Postman UI: workspace exists with `lending` naming
- [ ] Postman UI: spec in Spec Hub, 3 collections, 3 environments, 1 monitor
- [ ] Multipart upload endpoint generated correctly (verify in Postman UI)
- [ ] mTLS gap logged in `issues-log.md` even if run didn't fail on it
- [ ] `postman/collections/` contains JSON exports

**Commit message (final commit only):**

```
PR 1: Working onboarding workflow for loan-origination service

Structural copy of payments workflow; only per-service inputs changed:
- project-name, domain, domain-code, requester-email
- spec-url, environments-json, env-runtime-urls-json
- ci-workflow-path (collision avoidance)

First green run: <link>

Verified deltas: <N> input lines. Workflow shape identical to companion.

Refs: BLUEPRINT.md §4 PR 1, companion's PR 1
```

#### Execute prompt for PR 1

```
Execute PR 1 (Workflow file + first green run) from BLUEPRINT.md.

FILL IN BEFORE PASTING:
- YOUR_EMAIL: <your email>
- YOUR_POSTMAN_USER_ID: <numeric ID>
- OWNER: <your github username or org>

Context:
- PR 0 merged.
- Both secrets set on this repo (verify: `gh secret list`).
- Companion repo's PR 1 is green.

Steps (per BLUEPRINT.md §4 PR 1):
1. Create branch `pr-1-workflow`.
2. Verify secrets: `gh secret list`. STOP if either is missing.
3. Fetch companion's workflow:
   gh api repos/<OWNER>/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml \
     --jq .content | base64 -d > /tmp/companion-onboard.yml
4. Copy to this repo: `cp /tmp/companion-onboard.yml .github/workflows/onboard.yml`
5. Change ONLY these inputs (per BLUEPRINT.md §4 PR 1 step 4):
   - project-name → loan-origination-service
   - domain → lending
   - domain-code → LND
   - requester-email → YOUR_EMAIL
   - workspace-admin-user-ids → YOUR_POSTMAN_USER_ID
   - spec-url → raw URL to this repo's spec on main
   - environments-json → '["prod","staging","dev"]'
   - env-runtime-urls-json → the three lending-api URLs from the spec
   - ci-workflow-path → .github/workflows/loan-origination-tests.yml
6. Diff: `diff /tmp/companion-onboard.yml .github/workflows/onboard.yml`. Confirm
   only ~6 input lines changed. Show me both the diff and the final file.
7. After my approval: commit, push.
8. /run-and-watch.
9. If failed: diagnose, propose smallest fix, get my approval, iterate. Log via /log-issue.
10. After green:
    - Open the generated baseline collection in Postman UI.
    - Find the POST /applications/{id}/documents request. Confirm it's defined
      as multipart/form-data with the file part correctly configured.
    - Log mTLS-not-auto-configured as a follow-up via /log-issue, regardless of
      whether the run failed on it (the spec doesn't declare mTLS in
      securitySchemes, so the action correctly doesn't auto-wire it).

Stop and confirm before declaring done. Walk acceptance gate (7 boxes).
```

---

### PR 2 — `ADAPTATION.md` (the deliverable)

**Goal:** A documented analysis of what transfers and what changes. **This is
the highest-leverage file in this repo.** Evaluators read this first.

**Prerequisites:** PR 1 merged in both repos. Observations from PR 1 in BOTH
repos available.

**File to create:** `ADAPTATION.md` at repo root.

**Required content:**

1. **One-paragraph thesis.** Lead with the punchline. Real numbers.

   > "The onboarding pattern from `jr-cse-payments-postman-onboarding` transfers to
   > Loan Origination with **[N]** lines of workflow YAML changed and **[M]**
   > manual follow-up items. The compute (ECS+ALB vs Lambda+API Gateway) and
   > the auth pattern (mTLS+JWT vs OAuth+JWT) are configuration, not structural
   > concerns, at the onboarding layer."

   Replace `[N]` with the actual `diff` line count from PR 1. `[M]` is the
   count of items in section 3 below.

2. **The diff matrix.** Three columns minimum: Dimension, Payments value, Loan
   Origination value, Change type. Cover at minimum:

   | Dimension | Payments | Loan Origination | Change type |
   |---|---|---|---|
   | Compute target | Lambda + API Gateway | ECS + ALB | N/A (downstream of onboarding) |
   | Auth declared in spec `securitySchemes` | OAuth2 + JWT | JWT only | Config (env auth setup) |
   | Auth actually needed at runtime | OAuth2 client creds | mTLS + JWT | **Manual** (mTLS not in spec) |
   | Workflow file lines changed | n/a (baseline) | <N> input lines | Config |
   | Workflow structural changes | n/a | 0 | None |
   | Spec freshness | Current | Current | None |
   | Environments | prod/uat/qa/dev | prod/staging/dev | Config (per service) |
   | Server URL pattern | api.payments.example.com | lending-api.example.com | Config |
   | File-upload endpoints | None | POST /applications/{id}/documents (multipart) | **Validate generated collection** |
   | Generated workspace name | (observed) | (observed) | Auto-derived |
   | Generated collection count | 3 | 3 | None |
   | Monitor schedule | Default | Default | None |
   | `ci-workflow-path` | payments-tests.yml | loan-origination-tests.yml | Config (collision avoidance) |

   Populate "(observed)" rows from your PR 1-2 runs in BOTH repos.

3. **What needs follow-up.** Each item actionable, not vague.

   - **mTLS at runtime.** Spec's `securitySchemes` declares JWT bearer only;
     mTLS is mentioned in `info.description` text. Generated collection wires
     JWT correctly but does not configure client cert material. Pattern needed:
     pre-request script template referencing certs from a customer secret store.
     Customer ask: provision certs in their secret manager, reference in env var.

   - **Multipart upload validation.** `POST /applications/{id}/documents` uses
     `multipart/form-data`. Confirm the generated baseline collection has the
     file part correctly defined. If not, patch the collection post-generation
     (one-time per spec change) or surface as a contract-gen limitation.

   - **Environment URL realism.** All URLs are example.com. Customer ask: real
     URLs per env from the platform team.

   - **Monitor source-IP allowlisting.** mTLS endpoints are typically
     allowlist-only. Customer ask: allowlist Postman monitor source IPs.

   - (Add anything else from `issues-log.md`.)

4. **What this proves about the pattern.** Two paragraphs max.

   - Onboarding workflow is genuinely reusable: ~6 input lines is the diff
     between two services with different compute and different declared auth.
   - Follow-ups are honest scope: pattern works at catalog/spec/collection/env/
     monitor layer. Runtime concerns (mTLS, network restrictions, real
     credentials) are customer-side and live outside the onboarding workflow
     regardless of which service.

5. **Cross-reference.** Final paragraph: "See `<COMPANION_REPO_URL>` for the
   canonical pattern implementation, including the universal-vs-per-service
   matrix and the validation evidence."

**Acceptance gate:**

- [ ] `ADAPTATION.md` exists at repo root
- [ ] Thesis has real numbers, not placeholders
- [ ] Diff matrix has observed values for both services, not predictions
- [ ] Follow-up items are actionable (each names a customer-side action or a specific patch)
- [ ] Cross-link to companion repo present

**Commit message:**

```
PR 2: ADAPTATION.md — pattern transfers with <N> lines changed, <M> follow-ups

- Diff matrix populated from observed runs in both repos
- mTLS, multipart, env-URL gaps listed as actionable follow-ups
- Thesis: workflow shape unchanged; only config inputs differ

Refs: BLUEPRINT.md §4 PR 2, plan §4.3, brief Part 1 step 2
```

#### Execute prompt for PR 2

```
Execute PR 2 (ADAPTATION.md + diff matrix) from BLUEPRINT.md.

FILL IN BEFORE PASTING:
- COMPANION_REPO_URL: <https://github.com/<owner>/jr-cse-payments-postman-onboarding>

Context:
- PR 1 merged in BOTH repos.
- Observations from PR 1 in both repos available.

Steps (per BLUEPRINT.md §4 PR 2):
1. Run /compare-to-payments to build the raw diff between this repo and companion.
2. From the diff, populate ADAPTATION.md per BLUEPRINT.md §4 PR 2:
   - Thesis paragraph: count actual lines from the workflow file diff. Use real
     number for [N]. Count follow-ups for [M].
   - Diff matrix: fill EVERY row with observed values from BOTH repos. If a row
     is "predicted" rather than "observed", remove it.
   - Follow-ups: at minimum mTLS, multipart validation, env-URL realism,
     monitor source-IP allowlisting. Add anything else from issues-log.md.
3. Cross-reference: link to COMPANION_REPO_URL.
4. Read it aloud or summarize: does it sound like the document an evaluator
   wants to read first?
5. Commit per BLUEPRINT.md §4 PR 2 message template.
6. Confirm acceptance gate (5 boxes).

Do NOT pad. The brief grades on demonstrating how little changed. If 6 lines
changed, say 6.
```

---

### PR 3 — README (lean, cross-referenced)

**Goal:** Short README that points at `ADAPTATION.md` for analysis and at
companion repo for the canonical pattern.

**Prerequisites:** PR 2 merged.

**Sections to write:**

1. **Overview** — one paragraph: this is adaptation analysis for Loan
   Origination. Link to companion repo for canonical pattern.

2. **Why Loan Origination** — one short paragraph: max contrast with Payments
   on compute (ECS+ALB) and auth (mTLS+JWT). Pattern transfer is the deliverable.

3. **What's the Same vs What Changed** — point at `ADAPTATION.md`. Two-sentence
   summary; details in that file.

4. **Customer-Side Requirements (this service)** — focused list:
   1. Real environment URLs per env
   2. mTLS cert material in customer secret manager
   3. Monitor source IP allowlisting (network-restricted endpoints)
   4. JWT issuer/audience configuration

5. **Run Instructions** — short. Copy structure from companion repo's run
   instructions; note any deltas (likely none).

6. **Validation Evidence** — link to green run, link to committed
   `postman/collections/`, embed 1-2 screenshots of the workspace.

7. **Known Issues and Resolutions** — synthesize from `issues-log.md`. Likely
   shorter than companion's because pattern is now known.

8. **AI Assistance and Manual Validation** — template below.

**AI disclosure template for this repo:**

```markdown
## AI Assistance and Manual Validation

### What AI generated
- Copy of companion workflow file with per-service inputs swapped.
- Initial draft of ADAPTATION.md's structure (matrix headers, follow-up list shape).

### What I validated manually
- Diffed the two workflow files line by line to confirm only the expected
  inputs changed. Final delta: <N> lines.
- Inspected the generated multipart-upload request in the baseline collection
  to confirm the file part was correctly defined.
- Verified mTLS configuration is NOT auto-wired (consistent with spec's
  `securitySchemes` only declaring JWT — this is correct behavior, not a bug).

### What AI got wrong (and I corrected)
- AI proposed generating a new workflow from scratch rather than copying the
  companion's. Corrected: structural parity is the point of the pattern
  transfer story. Copy-then-diff is more honest than parallel-author-then-compare.
- (Add other specific corrections from your iterations.)

### Why this matters
The brief's strongest signal is "how little actually changes." Copy-then-diff
makes that signal measurable: <N> input lines, 0 structural changes. Parallel
authorship would have obscured that.

See <COMPANION_REPO_URL>'s AI disclosure for the canonical pattern's AI-vs-human
breakdown.
```

**Acceptance gate:**

- [ ] README is short (≤200 lines)
- [ ] Cross-links to companion repo and to `ADAPTATION.md` present
- [ ] AI disclosure references companion rather than restating

**Commit message:**

```
PR 3: README for loan-origination adaptation repo

- Lean: points at ADAPTATION.md for analysis, companion for pattern
- Customer-side requirements focused on mTLS + network restrictions
- AI disclosure references companion's canonical breakdown

Refs: BLUEPRINT.md §4 PR 3, plan §13, brief Part 1 step 3
```

#### Execute prompt for PR 3

```
Execute PR 3 (README) from BLUEPRINT.md.

FILL IN BEFORE PASTING:
- COMPANION_REPO_URL: <https://github.com/<owner>/jr-cse-payments-postman-onboarding>

Context:
- PR 2 merged. ADAPTATION.md is the principal artifact.
- Companion repo's README is canonical pattern reference.

Steps (per BLUEPRINT.md §4 PR 3):
1. Write README per BLUEPRINT.md §4 PR 3 sections 1-8.
2. Keep it short — target ≤200 lines.
3. Heavy use of cross-references: ADAPTATION.md for diff, COMPANION_REPO_URL for pattern.
4. AI disclosure (section 8): use template in BLUEPRINT.md §4 PR 3. Pull from
   THIS repo's iterations. Reference companion's AI disclosure rather than restate.
5. Show me the README before committing.
6. After my approval: commit per BLUEPRINT.md §4 PR 3 message template.
7. Confirm acceptance gate (3 boxes). After PR 3: repo URL ready to submit.

Anti-padding rule: if you find yourself restating what the action chain does or
re-explaining the pattern, STOP and link to companion's README instead.
```

---

## 5. `.claude/settings.json`

Same as companion's. Copy from companion BLUEPRINT §5.

---

## 6. Slash command files

Inherit three from companion (`log-issue`, `run-and-watch`, `validate-postman`).
Copy them verbatim from companion BLUEPRINT §6.

**Additional command unique to this repo:**

### `.claude/commands/compare-to-payments.md`

```markdown
Build or update the diff matrix in ADAPTATION.md by comparing this repo's state
to the companion payments repo.

Steps:
1. Ask me for COMPANION_REPO_URL if not in context (e.g., `<owner>/jr-cse-payments-postman-onboarding`).
2. For each file below, fetch companion's version via `gh api` and diff:
   - .github/workflows/onboard.yml
     `gh api repos/<owner>/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml --jq .content | base64 -d`
   - service.config.yml
     `gh api repos/<owner>/jr-cse-payments-postman-onboarding/contents/service.config.yml --jq .content | base64 -d`
   - README.md (sections that exist in both)
3. Categorize each change: Config / Structural / Manual / N/A.
4. Propose updates to the diff matrix in ADAPTATION.md based on findings.
5. Do NOT apply updates without my confirmation.

Output: a single proposed diff against ADAPTATION.md's matrix section.
```

---

## 7. `service.config.yml` template

```yaml
# service.config.yml — per-service onboarding inputs (no secrets)
#
# Documents inputs passed to postman-api-onboarding-action.
# Companion repo: jr-cse-payments-postman-onboarding (canonical pattern source)

service:
  name: loan-origination-service
  domain: lending
  domain_code: LND
  requester_email: <YOUR_EMAIL>

spec:
  path: specs/loan-origination-api-openapi.yaml

environments:
  - prod
  - staging
  - dev

environment_urls:
  prod:    https://lending-api.example.com/v1
  staging: https://lending-api-staging.example.com/v1
  dev:     https://lending-api-dev.example.com/v1

ci_workflow:
  generate: true
  path: .github/workflows/loan-origination-tests.yml

# Adaptation notes — what differs from the Payments baseline:
adaptation_notes:
  compute:
    payments: lambda + api gateway
    loan_origination: ecs + alb
    impact_on_onboarding: none (compute is downstream of catalog onboarding)

  auth_declared_in_spec:
    payments: oauth2 + jwt
    loan_origination: jwt
    impact_on_onboarding: env auth setup differs; one less secret needed

  auth_actually_needed_at_runtime:
    payments: oauth2 client credentials
    loan_origination: mtls + jwt
    impact_on_onboarding: |
      mTLS declared in spec info.description text only, not in securitySchemes.
      Generated collection wires JWT but not mTLS. Customer-side cert
      provisioning + pre-request script template needed.

  file_uploads:
    payments: none
    loan_origination: POST /applications/{id}/documents (multipart/form-data)
    impact_on_onboarding: |
      Validate generated baseline collection handles multipart correctly.
      Patch post-generation if not.

  network_restrictions:
    payments: not specified
    loan_origination: likely (mTLS endpoints typically allowlist-only)
    customer_ask: allowlist Postman monitor source IPs
```

---

## 8. `issues-log.md` format

Same format as companion. Append-only. Specific. Honest.

---

## 9. Acceptance gates summary

| PR | Gate |
|---|---|
| 0 | `tree` matches; `CLAUDE.md` cross-references companion (real URL, not placeholder); no secrets |
| 1 | Green run; workflow diff vs companion is only expected input lines (~6) |
| 2 | `ADAPTATION.md` has real numbers, observed values, actionable follow-ups |
| 3 | README is lean, cross-references companion, AI disclosure is specific |

---

## 10. Anti-patterns (avoid these)

- **Re-authoring the workflow file from scratch.** Copy from companion; change
  only per-service inputs. Pattern-transfer story is undermined by parallel
  authorship.
- **Padding ADAPTATION.md with restated pattern explanation.** This repo is
  the delta, not canonical reference. If you find yourself explaining the
  action chain, stop and link to companion's README.
- **Inflating the diff to look thorough.** Brief grades on demonstrating how
  little changed. If 6 lines changed, say 6.
- **Treating mTLS as a bug in the action.** Spec doesn't declare mTLS in
  `securitySchemes`. Action correctly follows spec. mTLS is customer-side
  config, not a tool bug.
- **Pushing to remote without showing the user the change first.**

---

## 11. When this BLUEPRINT is "done"

- [ ] PR 3 merged
- [ ] `ADAPTATION.md` is the document an evaluator wants to read first
- [ ] README is short and cross-references companion
- [ ] Diff matrix has real numbers
- [ ] AI disclosure has specific corrections

After that, deck and Q&A work happens in the candidate's claude.ai workspace.
