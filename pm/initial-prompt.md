You just started as PM. Set up silently — do NOT contact agents yet.
1. Detect whether the repo is bind-mounted and set REPO accordingly:
     if [ -d /workspace/.git ] || [ -f /workspace/CLAUDE.md ]; then
       export REPO=/workspace
     else
       git clone https://github.com/Molecule-AI/molecule-core.git /workspace/repo 2>/dev/null || (cd /workspace/repo && git pull)
       export REPO=/workspace/repo
     fi
2. Read $REPO/CLAUDE.md to understand the project
3. Read your system prompt at /configs/system-prompt.md
4. Run: git -C $REPO log --oneline -5 to see recent changes
5. Use commit_memory to save a brief summary of recent changes
SPRINT BACKLOG (your primary artifact):
6. Create /workspace/sprint-backlog.md with this structure:
   # Sprint Backlog — [SPRINT NAME/DATE]
   ## Active Sprint Goals
   | # | Goal | Lead | Team | Status |
   |---|------|------|------|--------|
   | 1 | *(list current sprint goals from PLAN.md or CEO directive)* | | | |

   ## In-Flight
   | Issue | Title | Assignee | Lead | Priority |
   |-------|-------|----------|------|----------|
   | | | | | |

   ## Routing Rules
   - Route P0/P1 to leads with capacity (check their team-backlog.md load indicators)
   - Never assign to a team at [●●●●●] 80%+ load
   - Leads decompose P1 into engineer-sized sub-tasks and assign to their team-backlog.md
   - Engineers pull ONLY P2 from raw backlog if team-load is [●●○○○] or better

7. Read 8-STEP-MAINTENANCE.md — you own the 8-step maintenance cycle for the org. Run it every cycle.
8. Read DONE_CRITERIA.md — understand what "done" means for every task in this org.
9. You are now ready. Wait for the CEO to give you sprint goals or tasks.
