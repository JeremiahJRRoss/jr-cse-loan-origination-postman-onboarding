# SETUP — jr-cse-loan-origination-postman-onboarding

Wake-up checklist. ~10 minutes if you've already done the companion (Payments) repo.
~30 minutes if starting fresh.

> **Prerequisite:** Companion repo `jr-cse-payments-postman-onboarding` should
> be set up first. This SETUP reuses its credentials, its PAT, and most of its
> tooling. If you're starting fresh without payments, do the companion's
> SETUP.md first — it has the full PAT creation flow, Postman CLI install, etc.

---

## Step 1 — Confirm reusable values from the payments build

If you completed the companion repo, these are already set in your shell or
your Postman account:

```bash
# All three should print "set"
echo "PMAK: ${POSTMAN_API_KEY:+set}${POSTMAN_API_KEY:-NOT SET}"
echo "Access token: ${POSTMAN_ACCESS_TOKEN:+set}${POSTMAN_ACCESS_TOKEN:-NOT SET}"
echo "User ID: ${POSTMAN_USER_ID:-NOT SET}"
echo "Owner: ${OWNER:-NOT SET}"
```

If any are NOT SET, re-export them per the companion's SETUP.md Step 1.
Pay particular attention to `POSTMAN_ACCESS_TOKEN` — it's session-scoped
and may have expired since you set up Payments. If unsure, re-extract:

```bash
postman login
export POSTMAN_ACCESS_TOKEN="$(cat ~/.postman/postmanrc | jq -r '.login._profiles[].accessToken')"
```

---

## Step 2 — Update your existing PAT to include this repo

The PAT you created for Payments has a repository access list scoped to the
old repo name. Add this repo to the list.

> 💡 **Why this is required:** Fine-grained PATs are name-scoped at the GitHub
> API level. If you skip this, the workflow will fail with a 403 on the
> commit-back step. This was iteration #5 during the Payments build — same
> failure mode applies here.

```bash
open "https://github.com/settings/personal-access-tokens"
```

1. Click your `postman-onboarding-workflow-write` token
2. Scroll to **Repository access** → "Selected repositories"
3. Click "Select repositories" → add `jr-cse-loan-origination-postman-onboarding`
4. Click **Update** at the bottom

The token value stays the same. Verify it can see the new repo (after you
create it in Step 3):

```bash
# Run this AFTER Step 3 (after the GitHub repo exists)
read -s PAT_CHECK
# Paste token, press Enter
curl -sS -H "Authorization: Bearer $PAT_CHECK" \
  https://api.github.com/repos/$OWNER/jr-cse-loan-origination-postman-onboarding | jq '{name, permissions}'
unset PAT_CHECK
```

Expected: `permissions.push: true`.

---

## Step 3 — Upload and create the GitHub repo

### 3.1 Unzip and prepare

```bash
cd ~/Desktop
# If a previous attempt exists, remove it first:
rm -rf jr-cse-loan-origination-postman-onboarding loan-origination-postman-onboarding
unzip jr-cse-loan-origination-postman-onboarding.zip
cd jr-cse-loan-origination-postman-onboarding
ls -la                    # confirm dotfiles present
```

### 3.2 Initialize and push

```bash
git init -b main
git add -A
git commit -m "Initial scaffold: adaptation repo with all 5 verified fixes pre-applied"
gh repo create jr-cse-loan-origination-postman-onboarding --public --source=. --push
```

**If `gh repo create` complains** ("Unable to add remote 'origin'" or "Name already exists"):

```bash
git remote set-url origin https://github.com/$OWNER/jr-cse-loan-origination-postman-onboarding.git 2>/dev/null \
  || git remote add origin https://github.com/$OWNER/jr-cse-loan-origination-postman-onboarding.git
gh repo create jr-cse-loan-origination-postman-onboarding --public 2>/dev/null
git push -u origin main
```

### 3.3 Verify

```bash
gh api repos/$OWNER/jr-cse-loan-origination-postman-onboarding/contents/.github/workflows | jq '.[].name'
# Expected: ["onboard.yml"]

gh api repos/$OWNER/jr-cse-loan-origination-postman-onboarding/contents/specs | jq '.[].name'
# Expected: ["loan-origination-api-openapi.yaml"]
```

---

## Step 4 — Configure secrets and variables

Same three secrets + two variables as the companion. You're reusing the same
values; the secrets just need to be set on this new repo too.

```bash
echo "$POSTMAN_API_KEY"      | gh secret set POSTMAN_API_KEY      --repo $OWNER/jr-cse-loan-origination-postman-onboarding
echo "$POSTMAN_ACCESS_TOKEN" | gh secret set POSTMAN_ACCESS_TOKEN --repo $OWNER/jr-cse-loan-origination-postman-onboarding

# Re-set the PAT (the same token value you used for Payments)
read -s PAT_NEW
# Paste token, press Enter
echo "$PAT_NEW" | gh secret set GH_FALLBACK_TOKEN --repo $OWNER/jr-cse-loan-origination-postman-onboarding
unset PAT_NEW

gh variable set REQUESTER_EMAIL --repo $OWNER/jr-cse-loan-origination-postman-onboarding --body "your.email@example.com"
gh variable set POSTMAN_USER_ID --repo $OWNER/jr-cse-loan-origination-postman-onboarding --body "$POSTMAN_USER_ID"

# Verify all five are present
gh secret   list --repo $OWNER/jr-cse-loan-origination-postman-onboarding
gh variable list --repo $OWNER/jr-cse-loan-origination-postman-onboarding
```

Expected:
- Secrets (3): `POSTMAN_API_KEY`, `POSTMAN_ACCESS_TOKEN`, `GH_FALLBACK_TOKEN`
- Variables (2): `REQUESTER_EMAIL`, `POSTMAN_USER_ID`

> 💡 **Tip for production:** if your team uses an organization, set these as
> org-level secrets with `--visibility selected --repos` listing both repos.
> One configuration step instead of two. The presentation guide §4.5 has the
> commands.

---

## Step 5 — Run the onboarding workflow

```bash
gh workflow run onboard.yml --repo $OWNER/jr-cse-loan-origination-postman-onboarding
sleep 5
RUN_ID=$(gh run list --repo $OWNER/jr-cse-loan-origination-postman-onboarding --workflow=onboard.yml --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch "$RUN_ID" --repo $OWNER/jr-cse-loan-origination-postman-onboarding
```

Expected: green in ~40 seconds on first attempt. All five fixes from the
Payments build are pre-applied, so no iteration loop this time. The first
workflow_dispatch run is the proof of pattern transfer.

The same three benign annotations from Payments will appear on this green run:
1. Node.js 20 runtime deprecation (upstream — see README §7)
2. OpenAPI spec linting warnings (brief's spec — see README §7)
3. "Failed to invite requester: Only one role is supported" (you're already the
   workspace creator — benign)

These are documented in `README.md §7` as expected upstream behavior.

---

## Step 6 — Sync local working tree

The action commits the generated CI workflow and the git-sync YAML collections
(`postman/collections/`) back to `main`. Pull them down:

```bash
cd ~/Desktop/jr-cse-loan-origination-postman-onboarding
git pull --rebase
git log --oneline -5
ls postman/collections/
ls .github/workflows/
```

Expected:
- `git log` shows a commit by `Postman CSE <help@postman.com>` titled "chore: sync Postman artifacts and metadata"
- `postman/collections/` contains 3 git-sync YAML collections (Baseline, Contract, Smoke)
- `.github/workflows/` contains both `onboard.yml` AND `loan-origination-tests.yml`

> The single-file JSON exports in `postman/exports/` (`baseline.json`,
> `contract.json`, `smoke.json`) hold these same three collections, committed
> separately per the brief — not produced by the action's sync.

---

## Step 7 — Validate in the Postman UI

Same six checks as the companion, against the new `[LND]` workspace this time.

Open Postman:

```bash
open "https://go.postman.co/workspaces"
```

Find the workspace whose name begins with `[LND]` — likely `[LND] loan-origination-service`.

| # | Check | What you should see |
|---|---|---|
| 1 | Workspace exists | Sidebar entry, name starts with `[LND]` |
| 2 | Spec in Spec Hub | APIs section → "Lending Platform API - Loan Origination Service" v1.4.0 renders |
| 3 | Three collections | `[Baseline]`, `[Contract]`, `[Smoke]` loan-origination-service — each opens to show populated requests |
| 4 | Three environments | Environments section: `prod`, `staging`, `dev` |
| 5 | Mock server | Mock Servers section shows one mock |
| 6 | Monitor | Monitors section shows one monitor |

### Two loan-specific additional checks

1. **Multipart file upload endpoint.** Open the Baseline collection, find
   `POST /applications/{id}/documents`. Confirm the request body shows
   multipart form-data with a `file` part defined. This is the spec-specific
   detail that's worth noting in ADAPTATION.md follow-ups if the generator
   handled it differently than expected.

2. **JWT auth wired (not mTLS).** Open any authenticated request, check the
   Authorization tab. Expected: JWT bearer token placeholder. This is correct
   — the spec declares only JWT in `securitySchemes`. mTLS lives in the spec's
   description text and is wired via the action's `ssl-client-*` inputs
   (commented out in `onboard.yml`).

---

## Step 8 — Capture the measured diff for ADAPTATION.md

This is the headline number for the whole repo. Run it after both repos are
green:

```bash
cd ~/Desktop/jr-cse-loan-origination-postman-onboarding

# Pull companion's onboarding workflow for comparison
gh api repos/$OWNER/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml \
  --jq .content | base64 -d > /tmp/companion-onboard.yml

# The full diff
diff /tmp/companion-onboard.yml .github/workflows/onboard.yml

# The headline number — count of changed lines (not context)
diff /tmp/companion-onboard.yml .github/workflows/onboard.yml | grep -c '^<'
```

Write that number down. Replace the `<N>` placeholders in `ADAPTATION.md` §1
and §3 with the real value.

---

## Step 9 — Capture validation evidence

Take five screenshots in the new `[LND]` workspace using `Cmd+Shift+4` →
space → click window:

1. `workspace-home.png` — sidebar showing all artifacts
2. `spec-hub.png` — Loan Origination spec rendered
3. `baseline-collection.png` — POST `/applications/{id}/documents` expanded
   (the multipart endpoint — the visually distinctive shot)
4. `environments.png` — three environments with lending-api URLs
5. `monitor.png` — the monitor's status page

```bash
mkdir -p ~/Desktop/jr-cse-loan-origination-postman-onboarding/docs/screenshots
mv ~/Desktop/workspace-home.png ~/Desktop/spec-hub.png ~/Desktop/baseline-collection.png \
   ~/Desktop/environments.png ~/Desktop/monitor.png \
   ~/Desktop/jr-cse-loan-origination-postman-onboarding/docs/screenshots/

cd ~/Desktop/jr-cse-loan-origination-postman-onboarding
git add docs/screenshots/
git commit -m "docs: validation evidence screenshots"
git push
```

Update README §6 with the new workspace URL and run URL.

---

## Step 10 — Fill in ADAPTATION.md

This is the principal artifact in this repo. Open `ADAPTATION.md` and replace
the `<N>` placeholders in §1 (thesis) and §3 (matrix). Add any loan-specific
findings (e.g., multipart endpoint observation, mTLS verification).

```bash
cd ~/Desktop/jr-cse-loan-origination-postman-onboarding
sed -i '' "s/<N>/<your-line-count>/g" ADAPTATION.md
git add ADAPTATION.md
git commit -m "docs: ADAPTATION.md — pattern transfers with <N> lines changed"
git push
```

---

## Troubleshooting

All the failure modes documented in the companion's SETUP.md `Troubleshooting`
section apply here identically. If you hit:

- **`variables: write` parse error** — should not happen; pre-fixed in this scaffold
- **"spec-url required"** — should not happen; pre-fixed
- **"refusing to allow... without `workflows` permission"** — likely PAT scope; see Step 2
- **403 on push** — your PAT didn't get this repo added; redo Step 2
- **Workspace empty / partial in Postman** — POSTMAN_ACCESS_TOKEN missing or expired; re-extract per Step 1
- **Duplicate collections / specs after multiple runs — expected, not a failure.** The onboarding action reuses the workspace (git-sync link) but re-creates the spec, collections, mock, and monitor with new IDs on every run, so repeated runs accumulate duplicates in the same workspace. This is the same behavior characterized in the companion repo (`README §10`). To reset: delete the extra objects, or nuke and recreate the workspace, then run once. Treat onboarding as create-once per service (see companion SETUP for the full cleanup recipe).

---

## What "done" looks like

- [ ] All steps 1-10 completed
- [ ] Six Postman UI checks passed + two loan-specific checks (Step 7)
- [ ] Five validation screenshots committed
- [ ] ADAPTATION.md `<N>` placeholders filled with real line counts
- [ ] Companion repo (Payments) is also green and validated

Once both repos are submission-ready, you have the full pattern-transfer
story: canonical pattern (Payments) + measured adaptation (Loan Origination),
both running end-to-end on real Postman workspaces.
