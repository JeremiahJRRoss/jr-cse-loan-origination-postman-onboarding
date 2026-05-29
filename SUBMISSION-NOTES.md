# Submission notes — Loan Origination (read me first)

A one-screen reading guide, in my own words. Every claim below points to a real
file in this repo.

## What this repo is

This is the **adaptation** half of the take-home. The canonical onboarding
pattern lives in the companion repo,
[`jr-cse-payments-postman-onboarding`](https://github.com/JeremiahJRRoss/jr-cse-payments-postman-onboarding)
(read that first for the full pattern + AI disclosure). This repo onboards a
**second, deliberately different service** — Loan Origination, which runs on
**ECS + ALB** (vs Payments' Lambda + API Gateway) and uses **mTLS + JWT** (vs
OAuth + JWT) — and shows that onboarding it changes **only 8 input values** in
`onboard.yml`. Nothing structural. The consulting narrative (cohorts, rollout,
the GitLab-CI team) lives in the deck; this repo is the evidence under it.

## What changed vs. what didn't

The full, measured analysis is in **[`ADAPTATION.md`](ADAPTATION.md)** — the
principal artifact; read it before the README. One-line thesis: *if onboarding
works for two services with different compute **and** different auth by changing
only 8 per-service inputs (`project-name`, `domain`, `domain-code`, `spec-url`,
`spec-path`, `environments-json`, `env-runtime-urls-json`, `ci-workflow-path`),
the pattern scales.* The workflow file is otherwise a structural copy of the
companion's (`ADAPTATION.md` §2 diff matrix, §3 measured diff).

## The two judgment calls

1. **mTLS is customer-side config, not a workflow change.** The spec declares
   only JWT in `securitySchemes`; mTLS appears in `info.description` prose only
   (transport-layer, terminated at the ALB). So mTLS required **zero** change to
   the onboarding workflow and is **not one of the 8 inputs** — it's a
   trust-store concern the ops team provisions. Evidence:
   `specs/loan-origination-api-openapi.yaml` + [`ADAPTATION.md`](ADAPTATION.md) §4.1.
2. **The multipart upload endpoint was validated, not assumed.** Loan
   Origination has a `multipart/form-data` upload endpoint that Payments
   doesn't. I checked the generated Baseline collection
   (`postman/exports/baseline.json`) and confirmed it wires the multipart body
   correctly (`file` + `documentType` parts). The universal pattern absorbed the
   new endpoint shape with no structural change. Evidence:
   [`ADAPTATION.md`](ADAPTATION.md) §4.2 + README §7.C.

## AI usage

Full disclosure is in [`README.md`](README.md) §8. Short version: AI drafted the
initial workflow file (copy-then-diff from the companion, not re-authored),
`ADAPTATION.md`, and this README. I validated the substance by **running it** —
diffing the two workflows line by line, inspecting the generated multipart
request, and confirming JWT-not-mTLS auth wiring. The five companion iteration
fixes were pre-applied (see `issues-log.md`).

## Known trade-offs

The generated CI test workflow (`.github/workflows/loan-origination-tests.yml`)
ships from `postman-repo-sync-action` as a single escaped line, so the Actions
tab flags it "Invalid workflow file." It's an upstream quirk that **regenerates
on every run**, so it's left as-generated and documented (a hand-fix or move is
clobbered next run) — same decision as the companion. Details: README §6 callout
and §7.C, plus `issues-log.md`.
