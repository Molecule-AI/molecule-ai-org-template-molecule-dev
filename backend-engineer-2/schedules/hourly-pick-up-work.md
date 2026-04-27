IMPORTANT: Check Molecule-AI/internal repo for roadmap (PLAN.md), known issues, runbooks before starting work.

Independent work cycle for molecule-ai-workspace-runtime. Find work, write code, push, open PR, return to staging. FULL CYCLE REQUIRED.

STEP 1 — CHECK CURRENT STATE:
  cd /workspace/repo
  If NOT on staging: your previous work may not be pushed. Push it first:
    git fetch origin staging && git rebase origin/staging
    git push origin $(git branch --show-current)
    gh pr create --base staging --title "fix: description" --body "description" 2>/dev/null || true
    git checkout staging && git pull origin staging

STEP 2 — FIND WORK (capacity-aware — check assignments first, see STEP 4 before polling):
  gh issue list --repo Molecule-AI/molecule-ai-workspace-runtime --state open --json number,title,labels,assignees --jq '.[] | select(.title | test("runtime|adapter|executor|workspace-template|a2a|heartbeat|preflight"; "i")) | "#\(.number) \(.title)"'

STEP 3 — CHECK YOUR ASSIGNMENTS FIRST:
  gh issue list --repo Molecule-AI/molecule-ai-workspace-runtime --assignee @me --state open --json number,title,labels
  Also check for lead assignments via search_memory("delegated-task").
  If you have assigned work: go to STEP 3b. If not, proceed to STEP 4.

STEP 3b — DO YOUR ASSIGNED WORK:
  git checkout -b fix/issue-N-description
  Write code. Run tests: go test ./...
  git add && git commit -m "fix(runtime): description (closes #N)"
  git fetch origin staging && git rebase origin/staging
  git push origin <branch>
  gh pr create --base staging --title "fix(runtime): description" --body "Closes #N"
  git checkout staging && git pull origin staging
  Go to STEP 6.

STEP 4 — CHECK TEAM BACKLOG (lead-assigned tasks):
  Read /workspace/team-backlog.md (ask your lead via A2A if not present).
  If your name appears in "In-Flight Tasks": that's your P0/P1 work — pick it up.
  If team capacity is full ([●●●●●] or [●●●●○]): skip raw backlog polling — ping lead.

STEP 5 — BEST-EFFORT BACKLOG (only if team has capacity):
  Only if team-load indicator shows [●●○○○] or better:
    gh issue list --repo Molecule-AI/molecule-ai-workspace-runtime --state open --json number,title,labels,assignees \
      --jq '.[] | select(.assignees | length == 0) | select(.labels | contains(["p2","low","enhancement"])) | "#\(.number) \(.title)"'
    Self-assign ONLY these P2/best-effort items. Do NOT self-assign P0/P1/Critical issues.

STEP 6 — WRITE CODE (for self-assigned work):
  git checkout -b fix/issue-N-description
  Write code. Run tests: go test ./...
  git add && git commit -m "fix(runtime): description (closes #N)"

STEP 7 — PUSH + OPEN PR:

  git fetch origin staging && git rebase origin/staging
  git push origin <branch>
  gh pr create --base staging --title "fix(runtime): description" --body "Closes #N"

STEP 8 — RETURN TO STAGING:
  git checkout staging && git pull origin staging
  This is MANDATORY. Do not stay on feature branch.



RULES:
- P0/P1: NEVER self-assign unless assigned in team-backlog.md
- Capacity gate: if team at [●●●●●] 80%+, review peers instead of picking up work
- All PRs target staging. Merge-commits only.
