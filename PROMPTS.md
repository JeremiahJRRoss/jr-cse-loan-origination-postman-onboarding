# Loan Origination Repo — Sequential Claude Code Prompts

> **Companion to:** `BLUEPRINT.md` (the spec). This file is the turn-by-turn
> operational guide. Each turn is one paste into `claude`.
>
> **Premise of this repo:** it's the **adaptation analysis**. The canonical
> pattern lives in `jr-cse-payments-postman-onboarding`. The work here is copy-then-diff,
> not re-authorship. The brief grades on demonstrating how little actually changes.
>
> **How to use:**
> 1. Complete pre-flight setup below.
> 2. Open the repo in Claude Code (`claude` from the repo root).
> 3. Work through turns in order. Each turn has: paste-able prompt, what to
>    expect back, and what to decide before the next turn.
> 4. If something goes sideways, see §Recovery prompts at the end.

---

## Pre-flight (do this before opening Claude Code)

| # | Step | Verify |
|---|---|---|
| P1 | Companion repo `jr-cse-payments-postman-onboarding` exists, PR 1+ merged, latest run green | You can link the green run URL |
| P2 | Repo `jr-cse-loan-origination-postman-onboarding` created on GitHub, cloned locally | `git remote -v` correct |
| P3 | Both secrets set on THIS repo (or at org level visible to this repo):<br>`gh secret list` shows POSTMAN_API_KEY and POSTMAN_ACCESS_TOKEN | Both visible in output |
| P4 | `CLAUDE.md` and `BLUEPRINT.md` committed to this repo | Files visible on GitHub |
| P5 | Capture these for substitution into prompts:<br>• YOUR_EMAIL<br>• YOUR_POSTMAN_USER_ID<br>• OWNER (your GitHub username or org)<br>• COMPANION_REPO_URL (e.g., `https://github.com/<owner>/jr-cse-payments-postman-onboarding`) | All four written down |

Org-level secret shortcut (one-time, both repos):

```bash
gh secret set POSTMAN_API_KEY --org <org> --visibility selected \
  --repos jr-cse-payments-postman-onboarding,jr-cse-loan-origination-postman-onboarding
gh secret set POSTMAN_ACCESS_TOKEN --org <org> --visibility selected \
  --repos jr-cse-payments-postman-onboarding,jr-cse-loan-origination-postman-onboarding
```

If P1-P5 aren't all done, fix that first.

---

## PR 0 — Scaffolding (with adaptation framing)

Goal: lay down static files; ensure `CLAUDE.md` cross-references the companion
repo with a real URL (not the placeholder).

### Turn 0.1 — Create scaffolding, link companion, show tree

Paste (substitute first):

```
Turn 0.1 — Scaffold the repo per BLUEPRINT.md §4 PR 0.

Inputs:
- COMPANION_REPO_URL: <https://github.com/<owner>/jr-cse-payments-postman-onboarding>

Steps:
1. Fetch the spec from the brief:
   mkdir -p specs
   curl -fsSL -o specs/loan-origination-api-openapi.yaml \
     https://raw.githubusercontent.com/postman-cs/cse-exercise/main/specs/loan-origination-api-openapi.yaml
   Verify it's non-empty and starts with `openapi: 3.0.3`.

2. Create everything else in the §4 PR 0 file tree:
   - .gitignore (same patterns as companion — node + OS + env)
   - .claude/settings.json (same as companion BLUEPRINT §5)
   - .claude/commands/log-issue.md (same as companion BLUEPRINT §6)
   - .claude/commands/run-and-watch.md (same as companion BLUEPRINT §6)
   - .claude/commands/validate-postman.md (same as companion BLUEPRINT §6)
   - .claude/commands/compare-to-payments.md (BLUEPRINT.md §6 — this repo only)
   - .github/workflows/.gitkeep (empty)
   - postman/collections/.gitkeep (empty)
   - service.config.yml (BLUEPRINT.md §7 — with the adaptation_notes section)
   - issues-log.md (same format as companion)
   - README.md stub (BLUEPRINT.md §4 PR 0 step 8)

3. Update CLAUDE.md: replace the `<COMPANION_REPO_URL>` placeholder near the top
   with my actual COMPANION_REPO_URL above. Show me the resulting line so I can
   verify.

4. Run `tree -a -I '.git'` and show me the output.

5. STOP. Do NOT commit or push yet.
```

**Expect:** All files created, `CLAUDE.md` placeholder replaced, tree printed.

**Decide:** Tree matches BLUEPRINT.md §4 PR 0 layout? Spec verbatim from brief?
CLAUDE.md placeholder actually replaced (not still `<COMPANION_REPO_URL>`)?
No secret values in any file?

---

### Turn 0.2 — Commit and push

Paste:

```
Turn 0.2 — Tree reviewed; proceed to commit.

Steps:
1. `git add -A`
2. Commit with the message template from BLUEPRINT.md §4 PR 0.
3. Push to origin/main.
4. Walk me through the 4-box acceptance gate in BLUEPRINT.md §4 PR 0.
5. STOP after acceptance gate confirmed.
```

**Expect:** Commit hash, push success, acceptance gate walked.

**Decide:** PR 0 done. Move to PR 1.

---

## PR 1 — Workflow file + first green run (structural copy with deltas)

Goal: copy companion's workflow file; change ONLY per-service inputs. Diff should
be ~6 lines. Anti-pattern: re-authoring from scratch.

### Turn 1.1 — Verify environment and fetch companion's workflow

Paste (substitute first):

```
Turn 1.1 — Verify secrets and pull companion's workflow as the structural baseline.

Inputs:
- OWNER: <your github username or org>

Steps:
1. `gh secret list`. Confirm POSTMAN_API_KEY and POSTMAN_ACCESS_TOKEN both
   present (either repo-level or org-level visible). STOP and report if either
   is missing.

2. Verify companion has a green run:
   gh run list --repo <OWNER>/jr-cse-payments-postman-onboarding --workflow=onboard.yml --limit 1
   Confirm the latest run status is "success". STOP if not — companion must be
   green before this repo starts.

3. Fetch companion's workflow file:
   mkdir -p .github/workflows
   gh api repos/<OWNER>/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml \
     --jq .content | base64 -d > /tmp/companion-onboard.yml
   Confirm file is non-empty and starts with `name:`.

4. Show me the first 50 lines of /tmp/companion-onboard.yml so I can verify it's
   the right file.

STOP and wait.
```

**Expect:** Secrets confirmed, companion run is green, companion workflow fetched.

**Decide:** Companion workflow looks right? Proceed to Turn 1.2.

---

### Turn 1.2 — Apply per-service deltas

Paste (substitute first):

```
Turn 1.2 — Apply per-service deltas. COPY-THEN-DIFF — do NOT re-author.

Inputs:
- YOUR_EMAIL: <your email>
- YOUR_POSTMAN_USER_ID: <your numeric Postman user ID>
- OWNER: <your github username or org>

Steps:
1. Create branch: `git checkout -b pr-1-workflow`

2. Copy companion's file as the starting point:
   cp /tmp/companion-onboard.yml .github/workflows/onboard.yml

3. Change ONLY these inputs (NOT the structure, NOT the order, NOT anything else):
   - project-name → loan-origination-service
   - domain → lending
   - domain-code → LND
   - requester-email → YOUR_EMAIL (likely same as companion; update if changed)
   - workspace-admin-user-ids → YOUR_POSTMAN_USER_ID (likely same as companion)
   - spec-url → https://raw.githubusercontent.com/<OWNER>/jr-cse-loan-origination-postman-onboarding/main/specs/loan-origination-api-openapi.yaml
   - environments-json → '["prod","staging","dev"]'
   - env-runtime-urls-json → the three lending-api URLs from the spec's servers block:
       prod:    https://lending-api.example.com/v1
       staging: https://lending-api-staging.example.com/v1
       dev:     https://lending-api-dev.example.com/v1
   - ci-workflow-path → .github/workflows/loan-origination-tests.yml
   - name (workflow name, top of file) → "Onboard Loan Origination Service to Postman"
   - push.paths in the on: trigger → 'specs/loan-origination-api-openapi.yaml'

4. Diff against the companion to show me exactly what changed:
   diff /tmp/companion-onboard.yml .github/workflows/onboard.yml
   Then count changed lines:
   diff /tmp/companion-onboard.yml .github/workflows/onboard.yml | grep -c '^[<>]'

5. Report:
   - Number of changed lines (raw diff line count)
   - List of inputs you actually modified
   - Anything NOT in the planned changes list above (should be zero)

6. Show me the full final .github/workflows/onboard.yml.

7. STOP. Do NOT commit. Wait for my review.

Anti-pattern check: if you find yourself adding fields that aren't in the
companion's file, STOP and ask me first. The pattern-transfer story dies if
this file diverges structurally.
```

**Expect:** Diff output, line count, full file, modifications list.

**Decide:** Diff is bounded (≤20 lines including context, ~6-10 changed lines
proper)? Nothing structural changed? Spec URL points to this repo? Approve.

---

### Turn 1.3 — Commit, push, watch first run

Paste:

```
Turn 1.3 — Workflow approved. Commit, push, run.

Steps:
1. `git add .github/workflows/onboard.yml`
2. Commit with message: "PR 1 WIP: workflow copy from companion with per-service deltas"
   (We'll squash to final message after green.)
3. Push the branch.
4. Either merge to main, or `gh workflow run onboard.yml --ref pr-1-workflow`
   (depends on trigger config). Tell me which path you took.
5. Run /run-and-watch.
6. Report:
   - Run ID
   - Status
   - If failure: paste the most relevant ~30 lines of failed log
   - If success: every output (workspace name/URL, spec ID, collection IDs,
     env IDs, monitor ID, mock URL)

STOP.
```

**Expect:** Run completes with status reported.

**Decide:** Green → skip to Turn 1.6. Failed → Turn 1.4.

---

### Turn 1.4 — Diagnose a failed run (use only if Turn 1.3 failed)

Paste:

```
Turn 1.4 — Diagnose the failed run.

Steps:
1. Pull full failure log: gh run view <RUN_ID> --log-failed
2. Identify root cause — be specific (which step, what error, what input).
3. Compare to companion's known-good config. Likely causes here, in order of
   probability:
   - Spec URL not reachable (wrong owner/repo/branch, file not on main yet)
   - environments-json malformed (must be valid JSON)
   - env-runtime-urls-json malformed (must be valid JSON)
   - workspace name collision with companion's workspace
   - Multipart endpoint in spec causing a generator error
4. Propose the SMALLEST possible fix. Do NOT apply yet.
5. /log-issue with: tried / error / proposed change / source of fix.
6. Report diagnosis + proposed fix. STOP.

If the cause looks like an upstream action bug rather than our config, say so —
we'd handle that differently (likely log as a known issue, not fix in repo).
```

**Expect:** Specific diagnosis, smallest fix, logged issue.

**Decide:** Approve fix, modify, or reject and re-diagnose.

---

### Turn 1.5 — Apply fix and re-run (paired with Turn 1.4)

Paste:

```
Turn 1.5 — Apply the approved fix.

Steps:
1. Apply the fix exactly as discussed. Show me the diff before committing.
2. Commit with a descriptive message (NOT a generic "fix").
3. Push.
4. /run-and-watch.
5. Report status.
6. STOP.

If still failed: I'll send Turn 1.4 again. Do NOT squash any commits.
```

**Expect:** Fix applied, new run status.

**Decide:** Green → Turn 1.6. Failed → back to Turn 1.4.

---

### Turn 1.6 — Post-green validation (multipart + mTLS)

Paste:

```
Turn 1.6 — Run is green. Service-specific validation.

Steps:
1. Walk me to the generated baseline collection in the Postman UI.
2. Find the request POST /applications/{id}/documents (the multipart file upload).
3. Open it and verify in the request body:
   - Body type: form-data (multipart)
   - There's a "file" part configured
   - The application/id parameter is correctly referenced from the path
4. Tell me what you see. Wait for my "ok" or for me to flag an issue.

5. Regardless of run outcome: /log-issue for the mTLS gap. Append this exact
   entry to issues-log.md (today's date):

   ### YYYY-MM-DD — mTLS not auto-configured in generated collection
   **Tried:** Reviewed generated baseline collection after green onboarding run.
   **Error:** Not a runtime error — a known limitation. The spec declares mTLS
   only in info.description text. securitySchemes declares JWT bearer only.
   The generated collection wires JWT correctly but does not configure client
   certificate material.
   **Changed:** Logged as a customer-side follow-up: cert material in customer
   secret manager + pre-request script template referencing certs by name.
   **Source of fix:** Inspection of upstream action behavior + spec analysis.
   This is correct action behavior given the spec, not a tool bug.

6. Run /validate-postman to walk me through the remaining artifacts (workspace,
   spec, other 2 collections, envs, monitor). Wait at each.

7. STOP after the full UI walkthrough.
```

**Expect:** Multipart check completed, mTLS gap logged, UI walkthrough done.

**Decide:** Was multipart generated correctly? If not, add it as an issue and a
follow-up in ADAPTATION.md (PR 2). Either way, mTLS goes in the follow-up list.

---

### Turn 1.7 — Finalize PR 1

Paste:

```
Turn 1.7 — Finalize PR 1.

Steps:
1. Confirm postman/collections/ has JSON files committed by repo-sync. List them.
2. Re-run the diff and confirm the final number of changed lines vs companion's workflow:
   gh api repos/<OWNER>/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml \
     --jq .content | base64 -d > /tmp/companion-onboard-final.yml
   diff /tmp/companion-onboard-final.yml .github/workflows/onboard.yml | grep -c '^[<>]'
   Report the number. This is the headline for PR 2's ADAPTATION.md thesis.
3. Compose final commit message per BLUEPRINT.md §4 PR 1 commit message template,
   filling in:
   - Number of changed lines from step 2
   - The green run URL
4. Decide squash vs keep iteration commits — recommend keeping (see companion's
   PR 1 rationale: brief evaluates on debugging documentation).
5. Walk me through the 7-box acceptance gate in BLUEPRINT.md §4 PR 1.
6. Merge to main.

STOP after merge.
```

**Expect:** Diff line count captured, acceptance gate walked, PR 1 merged.

**Decide:** PR 1 done. Capture the diff line count — you'll cite it in PR 2's
thesis paragraph.

---

## PR 2 — `ADAPTATION.md` (the deliverable)

Goal: the principal artifact. Evaluators read this first. Real numbers, observed
values, no padding.

### Turn 2.1 — Run /compare-to-payments to build raw diff

Paste (substitute first):

```
Turn 2.1 — Build raw diff data using /compare-to-payments.

Inputs:
- COMPANION_REPO_URL: <https://github.com/<owner>/jr-cse-payments-postman-onboarding>

Steps:
1. `git checkout -b pr-2-adaptation`
2. Run /compare-to-payments. Per the slash command's spec, fetch each file's
   companion version via `gh api` and diff:
   - .github/workflows/onboard.yml
   - service.config.yml
   - (skip README — both are still in flux)
3. For each file, categorize changes: Config / Structural / Manual / N/A.
4. Build a raw observation summary I can use as input for ADAPTATION.md.
   Include:
   - Workflow diff: exact line count + each changed input name
   - service.config.yml diff: count of changed values + which keys
   - List of observed Postman artifact deltas from PR 1 in this repo
     (workspace name, env count, mock URL, monitor schedule)
   - Compare to companion's PR 1 artifacts: what was the same, what differed
5. Report the raw observation summary. STOP.

Do NOT draft ADAPTATION.md yet. This turn is for gathering observed data.
```

**Expect:** Raw observation data ready for the thesis + matrix.

**Decide:** Are the numbers solid? Anything you need to verify in either repo
before drafting?

---

### Turn 2.2 — Draft ADAPTATION.md thesis + diff matrix

Paste:

```
Turn 2.2 — Draft ADAPTATION.md sections 1-2 (thesis + diff matrix).

Steps:
1. Create ADAPTATION.md at repo root.
2. Section 1 — Thesis (one paragraph, per BLUEPRINT.md §4 PR 2):
   - Use the REAL number of changed workflow lines from Turn 2.1.
   - Use the REAL count of follow-up items you're planning for section 3.
   - Lead with the punchline. No hedging. No preamble.
3. Section 2 — Diff matrix (per BLUEPRINT.md §4 PR 2 step 2):
   - Every row populated with OBSERVED values from both repos, not predicted.
   - If a row's value would have to be predicted rather than observed, DROP IT.
   - Cover the minimum rows listed in BLUEPRINT.md §4 PR 2 step 2.
   - Add any additional rows that surfaced in PR 1 here that aren't in the
     starter list.
4. Show me sections 1 and 2 only. STOP.

Anti-pattern check: if you find yourself explaining what the action chain does,
DELETE that — link to companion's README instead. This document is the delta.
```

**Expect:** Thesis with real numbers + matrix with observed values.

**Decide:** Does the thesis read like a punchline? Is every matrix row backed
by observation? Reject anything that smells predicted-not-observed.

---

### Turn 2.3 — Draft follow-ups + "what this proves" + cross-reference

Paste (substitute first):

```
Turn 2.3 — Draft ADAPTATION.md sections 3-5 (follow-ups, proof statement, cross-ref).

Inputs:
- COMPANION_REPO_URL: <https://github.com/<owner>/jr-cse-payments-postman-onboarding>

Steps:
1. Section 3 — What needs follow-up (per BLUEPRINT.md §4 PR 2 step 3):
   - At minimum: mTLS at runtime, multipart upload validation, env URL realism,
     monitor source-IP allowlisting.
   - Add anything else from issues-log.md.
   - Each item is ONE specific action — names what to do, who owns it (customer
     vs CSE), and what artifact it would produce. No vague follow-ups like
     "improve auth handling".
2. Section 4 — What this proves about the pattern (per BLUEPRINT.md §4 PR 2
   step 4):
   - Two paragraphs max.
   - First paragraph: the workflow shape is genuinely reusable; the diff line
     count proves it.
   - Second paragraph: follow-ups are honest scope — pattern works at the
     catalog/spec/collection/env/monitor layer; runtime concerns are
     customer-side and live outside any onboarding workflow regardless of
     which service.
3. Section 5 — Cross-reference (one short paragraph):
   - Link to COMPANION_REPO_URL.
   - Point at companion's README for the canonical pattern, universal-vs-
     per-service matrix, and validation evidence.
4. Show me sections 3, 4, 5. STOP.

Anti-pattern check: if section 4 says anything that's already in companion's
README, cut it.
```

**Expect:** Three sections drafted with concrete follow-ups + lean proof
statement.

**Decide:** Is each follow-up actionable? Are sections 4 and 5 short enough?
Cross-reference is a real working link?

---

### Turn 2.4 — Final review and commit PR 2

Paste:

```
Turn 2.4 — Finalize ADAPTATION.md and commit PR 2.

Steps:
1. Show me the COMPLETE ADAPTATION.md (all 5 sections together) as it'll be
   committed.
2. Read it as if you've never seen the repo. Flag:
   - Any number that's a placeholder (e.g., "N lines changed" should be the real
     number).
   - Any sentence that restates information from companion's README.
   - Any follow-up that's vague rather than actionable.
   - Any matrix row whose value you can't trace to an observation in either
     repo's PR 1.
3. Report flagged items. If any: STOP and wait for my approval to fix.
4. Once approved: `git add ADAPTATION.md`
5. Commit per BLUEPRINT.md §4 PR 2 commit message template, filling in:
   - N (changed workflow lines)
   - M (follow-up count)
6. Push.
7. Walk me through the 5-box acceptance gate in BLUEPRINT.md §4 PR 2.
8. Merge to main.

STOP after merge.
```

**Expect:** Final review with flagged items, commit, merge.

**Decide:** Is this the document an evaluator wants to read first? If not,
iterate before merging — this is the principal artifact for the brief's
"second-spec analysis" grading.

---

## PR 3 — README (lean, cross-referenced)

Goal: short README (≤200 lines). Points at ADAPTATION.md for the analysis.

### Turn 3.1 — Draft README sections 1-7

Paste (substitute first):

```
Turn 3.1 — Draft README sections 1-7. DO NOT COMMIT.

Inputs:
- COMPANION_REPO_URL: <https://github.com/<owner>/jr-cse-payments-postman-onboarding>

Steps:
1. `git checkout -b pr-3-readme`
2. Replace README.md (currently stub) with sections 1-7 per BLUEPRINT.md §4 PR 3:
   - Section 1 — Overview (one paragraph; link companion)
   - Section 2 — Why Loan Origination (one short paragraph; max contrast story)
   - Section 3 — What's the Same vs What Changed (two sentences max; link
     ADAPTATION.md for detail)
   - Section 4 — Customer-Side Requirements (4 items focused on THIS service:
     env URLs, mTLS certs, monitor source-IP allowlisting, JWT config)
   - Section 5 — Run Instructions (copy structure from companion's README;
     note any deltas, likely none)
   - Section 6 — Validation Evidence (link to green run, link to
     postman/collections/, embed 1-2 screenshots if you took any)
   - Section 7 — Known Issues and Resolutions (synthesize from issues-log.md;
     likely shorter than companion's because pattern is now known; mention
     mTLS gap explicitly)
3. Target line count: ≤200 lines for the WHOLE README (including section 8 to
   be added in Turn 3.2). After drafting, run `wc -l README.md` and report.
4. Show me sections 1-7. STOP.

Anti-pattern check: if you find yourself explaining what the action chain does
OR restating universal-vs-per-service patterns from companion, CUT that and
link to companion instead.
```

**Expect:** Seven short sections, line count reported.

**Decide:** Is it actually lean? Does it cross-reference rather than restate?
If over 150 lines already (before section 8), cut.

---

### Turn 3.2 — Draft section 8 (AI disclosure)

Paste (substitute first):

```
Turn 3.2 — Draft README section 8 (AI disclosure). DO NOT COMMIT.

Inputs:
- COMPANION_REPO_URL: <https://github.com/<owner>/jr-cse-payments-postman-onboarding>

Steps:
1. Read this repo's issues-log.md entirely.
2. Draft section 8 using the template in BLUEPRINT.md §4 PR 3 "AI disclosure
   template for this repo".
3. Specifics for this repo's AI disclosure:
   - "What AI generated" — at minimum the workflow copy + per-service input
     swaps + initial ADAPTATION.md structure.
   - "What I validated manually" — the diff against companion (line count),
     the multipart upload request inspection, the mTLS gap analysis.
   - "What AI got wrong" — at minimum 1 specific entry from issues-log.md. If
     the AI proposed re-authoring from scratch instead of copying, that goes
     here. Reference real iterations.
   - "Why this matters" — one short paragraph on copy-then-diff making the
     "how little changed" signal measurable.
   - Final line: link to companion's AI disclosure for the canonical pattern's
     AI-vs-human breakdown.
4. Show me section 8. STOP.

Anti-pattern check: every "what AI got wrong" entry must trace to a real
iteration in issues-log.md or to a specific decision documented above. If you
can't trace it, REMOVE it.
```

**Expect:** Section 8 with specific entries, links to companion's disclosure.

**Decide:** Every entry traceable? Length is short? Companion link works?

---

### Turn 3.3 — Final review, commit, submit

Paste:

```
Turn 3.3 — Finalize PR 3 and prepare for submission.

Steps:
1. Write sections 1-8 into README.md as one cohesive document.
2. `wc -l README.md` — confirm ≤200 lines. If over: identify what to cut and
   propose specific cuts. STOP for my approval if cuts needed.
3. Do a final read-through. Flag:
   - Anything aspirational rather than done
   - Any link that's broken (verify with `curl -I` if remote, `ls` if local)
   - Anything contradicting ADAPTATION.md
4. Report flagged items. If any: STOP for my approval to fix.
5. Once clean: commit per BLUEPRINT.md §4 PR 3 commit message template.
6. Push.
7. Walk me through the 3-box acceptance gate in BLUEPRINT.md §4 PR 3.
8. Merge to main.
9. Report:
   - Final repo URL
   - ADAPTATION.md URL (the principal artifact)
   - Total commits across PR 0-3
   - Any open follow-ups I should know about before submitting

STOP after report.
```

**Expect:** Lean README, all sections cohesive, PR 3 merged.

**Decide:** Both repos (this + companion) ready for submission?

---

## Recovery prompts (when things go sideways)

### R1 — Agent is re-authoring the workflow instead of copying

```
Stop. You are re-authoring the workflow from scratch.

This violates the pattern-transfer story. Reset:
1. `rm .github/workflows/onboard.yml`
2. Re-fetch companion's workflow:
   gh api repos/<OWNER>/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml \
     --jq .content | base64 -d > /tmp/companion-onboard.yml
3. cp /tmp/companion-onboard.yml .github/workflows/onboard.yml
4. NOW apply only the per-service input changes from Turn 1.2. Nothing else.
5. Diff and confirm only the per-service input lines changed.

The point of this repo is "how little actually changes." Parallel authorship
defeats that.
```

### R2 — Agent is padding ADAPTATION.md with pattern explanation

```
Stop. You are restating the canonical pattern. This belongs in companion's
README, not here.

Action:
1. Re-read BLUEPRINT.md §10 anti-patterns.
2. Identify every sentence in your current ADAPTATION.md draft that explains
   how the action chain works, what bootstrap does, what repo-sync does, or
   anything else that would also be true for the payments repo.
3. Cut all of it. Replace with: "See <COMPANION_REPO_URL> for the canonical
   pattern."
4. Show me the trimmed version.

ADAPTATION.md is the delta. If a sentence is true for both repos, it's not the
delta.
```

### R3 — Agent inflated the diff to look thorough

```
Stop. Your diff count doesn't match the file.

Re-verify:
1. Re-run: diff /tmp/companion-onboard.yml .github/workflows/onboard.yml
2. Count actual changed lines (not context lines):
   diff /tmp/companion-onboard.yml .github/workflows/onboard.yml | grep -c '^[<>]'
3. Halve that number (each change shows as both a < and a > line):
   The actual number of changed lines is grep -c '^<' (or '^>', they should match).
4. Report the real number.
5. If your earlier number was wrong, fix every citation of it in ADAPTATION.md.

The brief grades on demonstrating how little changed. Inflating the diff
undermines the entire repo's purpose.
```

### R4 — Agent is making up information

```
Stop. You appear to be generating from memory rather than the repo state.

Re-ground:
1. `pwd && git status && git log --oneline -10`
2. Re-read CLAUDE.md and BLUEPRINT.md fully.
3. Re-read issues-log.md fully.
4. Tell me what PR and turn we're on based on the repo state.
5. Wait for my next instruction.
```

### R5 — Agent treated mTLS as a bug

```
Stop. mTLS is not a tool bug.

Re-ground:
1. Re-read BLUEPRINT.md §10 anti-pattern #4 (mTLS).
2. The spec declares mTLS only in info.description text, NOT in
   securitySchemes. The action correctly follows the spec.
3. mTLS is a customer-side configuration concern — runtime cert provisioning
   plus a pre-request script template referencing certs by name. It belongs in
   ADAPTATION.md's follow-ups list, not in issues-log.md as a workflow failure.
4. If you've logged mTLS as a workflow failure, move it to ADAPTATION.md's
   follow-ups instead.
```

### R6 — Agent skipped a step or pushed without approval

```
Stop. You skipped a checkpoint.

Recovery:
1. `git log --oneline -10`
2. `git status`
3. If a commit was pushed that I didn't approve, do NOT revert without asking.
   Tell me what was done. I'll decide whether to revert or move on.
4. Re-read BLUEPRINT.md §10.
5. Wait.
```

### R7 — Agent is generating verbose responses

```
Stop. Your responses are too long.

New rule for the rest of this session:
- Lead with the answer in 1-3 sentences.
- Show output, then stop. No restating what you did.
- No preamble like "I'll now..." or "Let me...". Just do.
- Tables and code blocks only when actually useful.

Acknowledge with "ok" and continue.
```

---

## When you're done

After PR 3 is merged:

- [ ] Verify ADAPTATION.md is the document an evaluator would read first
- [ ] Verify README is lean and cross-references companion
- [ ] Run the §15.2 pre-submission checklist from the plan file
- [ ] Verify both repos (this + payments) are publicly accessible OR evaluator
      has been added as collaborator
- [ ] Capture both repo URLs for submission
- [ ] Switch context to the deck (lives outside both repos)

---

## Turn count summary

| PR | Turns | Cumulative |
|---|---|---|
| 0 | 2 | 2 |
| 1 | 5 base (1.1, 1.2, 1.3, 1.6, 1.7) + iteration loop (1.4↔1.5) | 7-9 |
| 2 | 4 | 11-13 |
| 3 | 3 | 14-16 |

**Target: 14-16 turns + recovery prompts as needed.**

This repo is intentionally smaller than companion (~23 turns) because:
- PR 1 is a structural copy, not original authorship
- PR 2 is one focused document
- PR 3 is intentionally lean

If you find yourself adding turns to "be thorough," you're probably padding.
The brief grades on demonstrating how little actually changes.
