IMPORTANT: Check Molecule-AI/internal repo for roadmap (PLAN.md), known issues, runbooks before starting work.

Independent work cycle for docs site. Find work, write content, push, open PR, return to main. FULL CYCLE REQUIRED.

STEP 1 — CHECK CURRENT STATE:
  cd /workspace/repo
  If NOT on main: push previous work first.
    git push origin $(git branch --show-current)
    gh pr create --base main --title "docs: description" --body "description" 2>/dev/null || true
    git checkout main && git pull origin main

STEP 2 — CHECK YOUR ASSIGNMENTS FIRST:
  gh issue list --repo Molecule-AI/molecule-app --assignee @me --state open --json number,title,labels
  Also check for lead assignments via search_memory("delegated-task").
  If you have assigned work: skip to "STEP 4 — DO YOUR ASSIGNED WORK". If not, continue.

STEP 2b — CHECK TEAM BACKLOG:
  Read /workspace/team-backlog.md (ask your lead via A2A if not present).
  If your name appears in "In-Flight Tasks": that's your P0/P1 work — proceed to STEP 4.
  If team capacity is full ([●●●●●] or [●●●●○]): skip raw backlog polling — ping lead instead.

STEP 2c — BEST-EFFORT BACKLOG (only if team has capacity):
  Only if team-load shows [●●○○○] or better:
    gh issue list --repo Molecule-AI/molecule-app --state open --json number,title,labels,assignees --jq '.[] | select(.title | test("app|frontend|UI|React|Next"; "i")) | "#\(.number) \(.title)"'
    Self-assign ONLY P2/best-effort items. Do NOT self-assign P0/P1/Critical.

STEP 3 — SELF-ASSIGN (P2 only after checking team-backlog):
  gh issue edit <NUMBER> --repo Molecule-AI/molecule-app --add-assignee @me
  If no team-backlog access OR team at capacity: do NOT self-assign. Stop here.

RULES:
- P0/P1: NEVER self-assign. Only work if in team-backlog.md with your name.
- Capacity gate: if [●●●●●] 80%+, review peers instead of picking up new work.
- All PRs target staging. Merge-commits only.
