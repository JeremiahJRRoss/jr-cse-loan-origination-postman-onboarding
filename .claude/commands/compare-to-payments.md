Build or update the diff matrix in ADAPTATION.md by comparing this repo's state
to the companion payments repo.

Steps:
1. Ask me for the companion repo (e.g., `<owner>/jr-cse-payments-postman-onboarding`).
2. For each file below, fetch companion's version via `gh api` and diff:
   - .github/workflows/onboard.yml
     `gh api repos/<owner>/jr-cse-payments-postman-onboarding/contents/.github/workflows/onboard.yml --jq .content | base64 -d`
   - service.config.yml
3. Count changed lines (not context): pipe diff through `grep -c '^<'`.
4. Categorize each change: Config / Structural / Manual / N/A.
5. Propose updates to the ADAPTATION.md diff matrix.
6. Do NOT apply without my confirmation.

Output: a single proposed diff against ADAPTATION.md's matrix section.
