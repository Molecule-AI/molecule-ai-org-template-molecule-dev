# Review Gates — Definition of "Ready to Merge"

Every PR must pass all applicable gates before a Lead merges it.
A Lead MUST NOT merge if any gate is missing, red, or waived without documentation.

---

## Gate 0 — Author Prerequisites (Lead verifies before starting gate review)

- [ ] PR targets `staging` (not `main`)
- [ ] PR is not a draft (`isDraft: false`)
- [ ] Author is a recognized agent identity (`[<role>-agent]` tag in PR body)
- [ ] PR has a linked issue (no orphan PRs)
- [ ] PR description explains WHAT changed and WHY, not just "fixes bug"

---

## Gate 1 — CI Pipeline (Automated)

```
gh pr checks <N>
```

- [ ] All CI checks green
- [ ] No cancelled checks (cancelled = superseded; rerun if needed)
- [ ] If checks fail: mechanical (lint/snapshot) → triage fixes on-branch.
  Real test failure → return to engineer.

---

## Gate 2 — Build (Automated)

- [ ] `go build ./...` passes (Go repos)
- [ ] `npm run build` passes (Node/TS repos)
- [ ] Docker image builds successfully (if Dockerfile changed)
- [ ] No new compile errors or type errors

---

## Gate 3 — Test Coverage (Automated + Manual)

```
# Run test coverage on changed files
cd <repo> && npm test -- --coverage 2>&1 | grep "All files"
# or: go test ./... -coverprofile=coverage.out && go tool cover -func=coverage.out
```

- [ ] All tests pass (`exit 0`)
- [ ] **Coverage threshold: ≥80% on changed files** (flag files with <70% as 🟡, <50% as 🔴)
- [ ] New code has corresponding unit tests (no new untested code paths)
- [ ] Regression tests added for bug fixes
- [ ] Integration tests added for behavior changes

**Coverage exception process:** If coverage gap is justified (e.g., generated code, config-only change), author must document WHY in the PR body. Lead may waive with `[lead-agent] WAIVE-COVERAGE: <reason>`.

---

## Gate 4 — Security Review (Security Auditor)

Required for ALL PRs. Security Auditor posts ONE of:

```
[security-auditor-agent] APPROVED — no security concerns
[security-auditor-agent] CHANGES REQUESTED: <specific finding>
[security-auditor-agent] N/A — docs/content-only change
```

Security review scope:
- SQL injection, path traversal, XSS, SSRF, command injection, YAML injection
- Auth boundary changes (new endpoints, changed permissions)
- Secrets handling (tokens in logs/responses/error messages)
- Secret comparisons using `ConstantTimeCompare` (not `==`)
- Credential storage at rest (must be encrypted, not plaintext)
- Dependency CVE check (`gh advisory database ls` or `npm audit`)
- CORS configuration (no `*` without justification)
- Rate limiting on new endpoints

**For auth/billing/schema/data-deletion PRs:** Security Auditor MUST run `cross-vendor-review` skill before posting APPROVED.

**SAST enforcement:**
```
# Run SAST scan
cd /workspace/repo
# Go: gosec or semgrep
semgrep --config=auto . 2>&1 | grep -v "Scanning" | head -50
# Python: bandit
bandit -r . -f json 2>/dev/null | jq '.results[] | select(.severity=="HIGH" or .severity=="MEDIUM")'
```
- [ ] No HIGH severity SAST findings
- [ ] MEDIUM findings documented with justification

---

## Gate 5 — Design & Philosophy Review (Lead + PM for cross-cutting)

Required for: any structural change, new API endpoint, architecture change, feature that changes user-facing behavior.

Lead verifies:
- [ ] Change fits the existing architecture (not a local optimum)
- [ ] Change doesn't violate the 4 philosophies in SHARED_RULES.md:
  - **Philosophy 1:** Does the fix address the *class* of bug, not just the symptom?
  - **Philosophy 2:** Were discoveries (side findings) filed as separate issues?
  - **Philosophy 3:** Does the status report show the iceberg, not just the tip?
  - **Philosophy 4:** Did the author check `internal/` before building?
- [ ] Change doesn't introduce inconsistency with existing shipped patterns
- [ ] Change is reversible (or rollback plan is documented)

For cross-cutting changes (affects multiple teams): PM also reviews.

```
[lead-agent] DESIGN REVIEW PASSED
[lead-agent] DESIGN CONCERNS: <list>
[pm-agent] PHILOSOPHY REVIEW: <pass/concerns>
```

---

## Gate 6 — Code Review (Lead or designated peer)

Invoke `code-review` skill. Any 🔴 blocks merge.

Review checklist:
- [ ] No trivially wrong lines (kwargs vs positional, nil vs empty slice)
- [ ] Error handling is present and specific
- [ ] No commented-out code left in
- [ ] No TODO comments without linked issues
- [ ] Logging is present for operations (not sensitive data)
- [ ] Input validation at API boundary
- [ ] No hardcoded credentials or tokens
- [ ] Function/docs comments are accurate
- [ ] Test assertions are meaningful (not just `t.Error(nil)`)

---

## Gate 7 — UI/UX Review (UIUX Designer, if canvas/UI changed)

Required for: any change touching `canvas/src/**`, `molecule-app/src/**`, or any user-facing component.

UIUX Designer posts:
```
[uiux-agent] APPROVED — visual check passed
[uiux-agent] CHANGES REQUESTED: <specific issue>
[uiux-agent] N/A — no UI/UX surface changed
```

Review scope:
- [ ] Dark theme compliance (no hardcoded white backgrounds)
- [ ] Accessibility (keyboard navigable, correct contrast ratios)
- [ ] Responsive at 1920×1080, 1280×720, 768×1024
- [ ] No native `alert()`/`confirm()` dialogs (use `ConfirmDialog` component)
- [ ] Playwright E2E test passes for changed flows

**Playwright E2E:**
```
cd canvas && npm test 2>&1 | tail -20
# Or run specific test:
npx playwright test --grep "<feature-name>"
```

---

## Gate 8 — Integration/E2E Test (Integration Tester)

Required for: any change to platform services (molecule-core, molecule-controlplane, tenant-proxy, workspace-runtime).

Integration Tester verifies:
- [ ] Full workspace lifecycle passes (create → provision → heartbeat → A2A → delete)
- [ ] No contract mismatches in API response shapes
- [ ] WebSocket protocol intact
- [ ] Scheduler cron fires correctly

```
# From integration-tester/schedules/e2e-test.md
# Run the full E2E flow — all steps must pass
```

---

## Gate 9 — Cross-Vendor Review (for auth/billing/schema/data-deletion)

Required for: any PR touching authentication, billing, schema migrations, data deletion, or that changes behavior across multiple vendors/platforms.

Invoke `cross-vendor-review` skill before Gate 4 approval.

```
[security-auditor-agent] CROSS-VENDOR REVIEW: <clear/concerns>
```

---

## Blast-Radius Classification

Before merging, Lead classifies blast radius:

| Class | Examples | Additional Requirements |
|-------|----------|----------------------|
| **Low** | Typo fix, lint, doc-only, test-only | Standard gates |
| **Medium** | New feature, API addition, config change | Gate 5 (Design) |
| **High** | Auth change, billing, schema migration | Gate 4 + Gate 9 + PM ack |
| **Critical** | Data deletion, secret rotation, multi-tenant isolation fix | Gate 4 + Gate 9 + CEO ack + rollback plan documented |

**PM must acknowledge** before Lead merges High or Critical PRs.
**CEO must acknowledge** before Critical PRs merge.

---

## Merge Decision Tree

```
All 9 gates pass?
├── YES → Lead merges (merge-commit only, --delete-branch)
├── NO (gate 1-3 automated) → Return to engineer to fix
├── NO (gate 4) → Security Auditor files issue, engineer fixes
├── NO (gate 5) → Lead + PM review, engineer reworks
├── NO (gate 6) → Lead or peer reviews, engineer addresses feedback
├── NO (gate 7) → UIUX Designer files issue, engineer fixes
├── NO (gate 8) → Integration Tester files issue, engineer fixes
└── NO (gate 9) → Hold for PM/CEO ack per blast-radius class
```

---

## Waive Process

A Lead may waive Gates 5–9 for **trivial changes only** (1-line typo, lint fix, doc formatting). Waiver must be documented:

```
[lead-agent] WAIVE-REVIEW: <gate name> — reason
```

Gates 1–3 (CI/Build/Tests) and Gate 9 (Cross-Vendor) CANNOT be waived.
