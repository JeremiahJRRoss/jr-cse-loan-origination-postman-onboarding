# Issues Log — jr-cse-loan-origination-postman-onboarding

> Honest documentation of issues encountered during build.
> Append-only. Each entry is dated and specific. This feeds the README's
> "Known Issues and Resolutions" and "AI Assistance" sections — both graded.
>
> This build inherited the companion's five verified fixes pre-applied
> (variables-write, spec-url, gh-fallback-token, PAT verification, PAT rename),
> so onboarding was one-shot — those iterations did not recur here. The
> companion (`jr-cse-payments-postman-onboarding`) `issues-log.md` documents
> them. Entries below are loan-specific only.

---

<!-- Add entries below as you build. Template:

### YYYY-MM-DD — <one-line summary>
**Tried:** <what I was attempting>
**Error:**
```
<exact error text — verbatim>
```
**Changed:** <what I modified>
**Source of fix:** AI / Docs / action.yml / experiment / other

-->

### 2026-05-29 — generated CI workflow committed as escaped single line (same as companion)
**Tried:** Inspecting `.github/workflows/loan-origination-tests.yml` after onboarding.
**Error:** One physical line, 90 literal `\n`, invalid YAML (`mapping values are not allowed here, line 1, column 49`); "Invalid workflow file" on GitHub. Original is 4370 bytes — byte-identical to the companion's broken `payments-tests.yml`.
**Changed:** Decoded `\n` → newlines; verified lossless against the committed bytes (4370 → 4280, reversible) and that it parses with `on` + `jobs.test`. Byte-identity check vs the companion's *fixed* file deferred — companion repo is out of scope for this session's GitHub access; run the `diff` locally.
**Source of fix:** experiment. Root cause documented in the companion's README §11.A #6 — upstream repo-sync escaping; same generic workflow → same bug.

### 2026-05-29 — decoded CI workflow still can't go green on clean checkout (.postman/ gitignored)
**Tried:** Reasoning through whether the now-valid `loan-origination-tests.yml` would pass after PR R1.
**Error:** Its "Resolve Postman Resource IDs" step does `YAML.load_file('.postman/resources.yaml')` and aborts when missing; `.postman/` is gitignored and untracked, and the spec environments target `example.com`.
**Changed:** Documented as a known limitation (README §7.C) — Option C, mirroring the companion's layer-2 decision. No un-ignore here. One-line customer fix: un-ignore `.postman/`.
**Source of fix:** mirror of companion decision (see companion blueprint §3 options table).

### 2026-05-29 — VERIFIED-NOTES.md carried two stale claims contradicting onboard.yml
**Tried:** Reconciling `VERIFIED-NOTES.md` against the committed `onboard.yml`.
**Error:** Claimed a `variables: write` permission (no such GitHub Actions key — parse error) and that the scaffold "uses spec-path" only, while `onboard.yml` provides both `spec-url` and `spec-path`.
**Changed:** Corrected both claims in place and demoted the header's authority line to point at `onboard.yml` as the source of truth. Chose correct-in-place (parity) over delete; companion's outcome unverifiable this session (out of scope).
**Source of fix:** verified against `onboard.yml` + action.yml ("GitHub token used for repo variables and generated commits").
