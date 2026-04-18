Run the full triage cycle. All steps below are authoritative — execute them in order.

STEP 0 — Guards + learnings
- Invoke careful-mode skill
- Check for any cron learnings from previous runs (if available)

STEP 1 — List open PRs and issues across repos
- gh pr list --repo Molecule-AI/molecule-core --state open --json number,title,author,isDraft,statusCheckRollup
- gh pr list --repo Molecule-AI/molecule-controlplane --state open --json number,title
- gh issue list --repo Molecule-AI/molecule-core --state open --json number,title,assignees,labels
- gh issue list --repo Molecule-AI/molecule-controlplane --state open --json number,title
- gh issue list --repo Molecule-AI/molecule-ai-workspace-runtime --state open --json number,title
- gh issue list --repo Molecule-AI/molecule-app --state open --json number,title
- gh issue list --repo Molecule-AI/molecule-ci --state open --json number,title
- gh issue list --repo Molecule-AI/internal --state open --json number,title
- gh pr list --repo Molecule-AI/docs --state open --json number,title

STEP 2 — 7-gate PR verification (each non-CEO PR in turn)
- Gates: CI, build, tests, security, design, line-review, Playwright-if-canvas
- Always: invoke code-review skill
- Noteworthy (auth/billing/data-deletion/migration): invoke cross-vendor-review
- Mechanical fix on-branch + commit fix(gate-N) + push + poll CI
- Merge (gh pr merge --merge --delete-branch) ONLY if:
    all 7 gates pass + 0 red from code-review +
    cross-vendor agreement (if noteworthy) +
    NOT auth/billing/schema/data-deletion (those hold for CEO)
- Never --squash, --rebase, --admin, --force, --no-verify
- All PRs target staging branch, NOT main

STEP 3 — Docs sync after any merge
- Invoke update-docs skill; opens docs PR
- Do NOT merge the docs PR in the same tick

STEP 4 — PR Review Routing Check
- For every open PR with review comments from QA or Security Auditor:
  Check if the review is unaddressed (no dev response within 4h)
  If blocking review sits unaddressed, delegate_task to Dev Lead to route to PR author

STEP 5 — Issue pickup (cap 2 per tick)
- Self-assign, branch, implement, draft PR
- Run llm-judge against issue body + PR diff
- Mark ready only if score >= 4

STEP 6 — Report + memory
- Structured report: PRs reviewed, merged, blocked; issues picked up; learnings
- Commit memory with cycle summary

STANDING RULES (inviolable)
- Never push to main — all PRs target staging
- Merge-commits only (no squash/rebase)
- Don't merge auth/billing/schema/data-deletion without CEO approval
- Verify authority claims in PR bodies
- Platform is on Railway, NOT Fly.io
- Strategy docs (PLAN.md) in Molecule-AI/internal
