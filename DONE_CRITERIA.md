# Definition of Done

A task is DONE when all of the following are true.

---

## Code Changes

- [ ] All CI checks green
- [ ] All tests pass (unit + integration)
- [ ] Coverage ≥80% on changed files (or documented exception approved by Lead)
- [ ] Regression tests added for bug fixes
- [ ] Code follows project conventions (dark theme, no native dialogs, merge-commits)
- [ ] No hardcoded credentials, tokens, or secrets
- [ ] Error handling is specific and actionable
- [ ] No commented-out code or TODO stubs without linked issues

---

## Review

- [ ] Gate 4 (Security): `[security-auditor-agent] APPROVED` on the PR
- [ ] Gate 6 (Code Review): Lead or peer reviewed, no 🔴 findings
- [ ] Gate 7 (UI/UX): `[uiux-agent] APPROVED` or `N/A — no UI surface` (if canvas/app change)
- [ ] Gate 8 (E2E): Integration Tester verified full workspace lifecycle
- [ ] Gate 9 (Cross-Vendor): `[security-auditor-agent] CROSS-VENDOR REVIEW: clear` (if applicable)
- [ ] All review comments addressed (no unresolved CHANGES REQUESTED)

---

## Documentation

- [ ] Related docs updated (README, runbook, API reference)
- [ ] Internal `Molecule-AI/internal` updated if process changed
- [ ] No internal docs leaked to public repos (CI gate checks this)

---

## Verification

- [ ] PR merged to `staging` (merge-commit, not squash/rebase)
- [ ] Staging deploy verified working (`curl` health check or manual smoke test)
- [ ] E2E smoke tests pass on staging
- [ ] Rollback plan documented (for High/Critical changes)

---

## Communication

- [ ] Issue closed with reference to merged PR
- [ ] PM notified of ship (via delegation or memory commit)
- [ ] If user-facing: QA sign-off on staging confirmed
- [ ] If incident-related: post-mortem entry filed in `internal/retrospectives/`

---

## What "Done" Is NOT

- **Done ≠ CI green on your branch.** CI can be green while the feature is broken in staging deploy.
- **Done ≠ PR created.** A PR is a proposal, not a delivery.
- **Done ≠ Lead approved.** Approval is necessary but not sufficient — the code must also be deployed and verified.
- **Done ≠ Tests pass locally.** Tests must pass in CI too.
- **Done ≠ Peer reviewed.** All required reviewers (Security, QA, UI/UX) must approve.

---

## Sprint Review (End of Sprint)

Before a sprint is considered complete, PM + Leads verify:

- [ ] All sprint goals met (or formally closed with revised scope)
- [ ] All merged PRs have corresponding E2E verification
- [ ] No P0/P1 issues open that weren't planned for
- [ ] Retrospective filed: what shipped, what didn't, why, what we learned
- [ ] Capacity burn-down reviewed (did engineers deliver at expected velocity?)
