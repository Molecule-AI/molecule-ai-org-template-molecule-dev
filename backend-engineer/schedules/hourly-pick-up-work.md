IMPORTANT: Check Molecule-AI/internal repo for roadmap (PLAN.md), known issues, runbooks before starting work.

Independent work cycle. Check team-backlog FIRST, then pick up work responsibly.

STEP 1 — PUSH ANY UNPUSHED WORK:
  cd /workspace/repo
  If NOT on staging: push any existing work first:
    git fetch origin staging && git rebase origin/staging
    git push origin $(git branch --show-current)
    gh pr create --base staging --title "fix: description" --body "description" 2>/dev/null || true
    git checkout staging && git pull origin staging

STEP 2 — CHECK YOUR ASSIGNMENTS FIRST:
  gh issue list --repo Molecule-AI/molecule-core --assignee @me --state open --json number,title,labels
  Also check for lead assignments via search_memory("delegated-task").
  If you have assigned work: go to STEP 3. If not, proceed to STEP 4.

STEP 3 — DO YOUR ASSIGNED WORK:
  git checkout -b fix/issue-N-description
  Write code. Run tests: cd workspace-server && go test -race ./...
  git add && git commit -m "fix(platform): description (closes #N)"
  git fetch origin staging && git rebase origin/staging
  git push origin <branch>
  gh pr create --base staging --title "fix(platform): description" --body "Closes #N"
  git checkout staging && git pull origin staging
  Go to STEP 6.

STEP 4 — CHECK TEAM BACKLOG (lead-assigned tasks):
  Read /workspace/team-backlog.md (ask Dev Lead via A2A if not present).
  If your name appears in "In-Flight Tasks": that's your P0/P1 work — pick it up.
  If team capacity is full ([●●●●●] or [●●●●○]): skip raw backlog polling — ping lead.

STEP 5 — BEST-EFFORT BACKLOG (only if team has capacity):
  Only if team-load indicator shows [●●○○○] or better:
    gh issue list --repo Molecule-AI/molecule-core --state open --json number,title,labels,assignees \
      --jq '.[] | select(.assignees | length == 0) | select(.labels | contains(["p2","low","enhancement"])) | "#\(.number) \(.title)"'
    Self-assign ONLY these P2/best-effort items. Do NOT self-assign P0/P1/Critical issues.

STEP 6 — RETURN TO STAGING:
  git checkout staging && git pull origin staging
  This is MANDATORY. Do not stay on feature branch.

RULES:
- All PRs target staging. Rebase before push. Merge-commits only.
- P0/P1 issues: NEVER self-assign. Only work them if your name is in team-backlog.md
- If team-load is [●●●●●] 80%+: do not pick up new work — help review peer PRs instead
- Report: commit_memory "work-cycle HH:MM - assigned #N / picked #M, tests pass/fail"

