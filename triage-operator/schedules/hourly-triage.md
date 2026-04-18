Run the full triage cycle. Execute all steps in order. Produce a report at the end.

STEP 1 — List open PRs and issues across repos:
  gh pr list --repo Molecule-AI/molecule-core --state open --json number,title,author,isDraft,statusCheckRollup
  gh pr list --repo Molecule-AI/molecule-controlplane --state open --json number,title
  gh issue list --repo Molecule-AI/molecule-core --state open --json number,title,assignees --jq '.[] | select(.assignees | length == 0) | "#\(.number) \(.title | .[0:60])"'
  gh issue list --repo Molecule-AI/molecule-controlplane --state open --json number,title

STEP 2 — For each non-CEO, non-draft PR with green CI: review the diff, check for bugs, post a review comment with [triage-agent] tag. If all gates pass, merge with gh pr merge --merge.

STEP 3 — Check for unaddressed review comments on open PRs. If QA or Security posted a blocking review >4h ago with no dev response, note it in the report.

STEP 4 — Pick up 1-2 unassigned issues if available. Self-assign, create branch from staging, implement, push, open PR targeting staging.

STEP 5 — Report what you did: PRs reviewed/merged, issues picked up, blockers found.

RULES: All PRs target staging. Merge-commits only. Never squash/rebase. Platform on Railway not Fly.io. PLAN.md in Molecule-AI/internal.
