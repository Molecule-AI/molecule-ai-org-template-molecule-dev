# 8-Step Maintenance Checklist

Runs every maintenance cycle. This is the autonomous team's health monitoring loop.

## STEP 1 — Output Check
Verify all agents are producing real output (not phantom busywork).

- Check cron-learnings.jsonl for any agent posting empty or nonsensical output
- Check any agent with active_tasks stuck > 0 for 3+ cycles (phantom-busy)
- Check for agents that have been SILENT for > 2 hours despite having hourly crons
- Check team-backlog.md files: are engineers actually shipping, or just claiming?

**Action:** If phantom-busy or silent agent found: delegate_task to their Lead to investigate.

## STEP 2 — GH_TOKEN Refresh
Tokens expire every 60 minutes. Proactively refresh before they expire.

- Check credential-token-backlog in Molecule-AI/internal
- If any token is > 55 minutes old: trigger refresh via the token refresh daemon
- Do NOT wait for a 401 to refresh — be proactive
- Document refresh events in credential-token-backlog

**Action:** If token refresh daemon is broken: file P1 issue, notify DevOps.

## STEP 3 — Unpushed Work
Check for any branches with commits that have not been pushed.

- gh api repos/Molecule-AI/<repo>/commits?per_page=5 — compare local vs remote
- For any unpushed work: notify the engineer via A2A, give them 10 minutes to push
- If no response: check if they are blocked on CI or need help

**Action:** Engineer has 10 minutes to push or explain. If silent > 10 min, Lead reassigns.

## STEP 4 — Review Routing
Ensure PR reviews are being assigned to the right people.

- Check PR queue: any PR waiting > 2h without a reviewer assigned?
- Check triage-operator: are they routing PRs to Security/QA/UIUX reviewers?
- Check for PRs with unaddressed CHANGES REQUESTED — author not responding?

**Action:** If review stalled > 2h: triage-operator pings the reviewer directly.

## STEP 5 — New Containers
Check for any new workspace containers that need onboarding.

- gh api /workspaces — any new workspaces since last cycle?
- For each new container: verify it has a role, a parent lead, and a first-task assigned
- If new container has no parent: assign to triage-operator for routing

**Action:** New containers should have work within 2 hours of creation.

## STEP 6 — Cron First-Fires
Verify scheduled jobs fired correctly in the last hour.

- Check scheduler liveness: curl -sf http://localhost:8080/admin/liveness
- Check recent cron fire timestamps — any cron that should have fired but did not?
- Check for crons that fired but produced no output (stuck prompts)

**Action:** Any cron missing a fire: check the cron expression, check the agent is alive.

## STEP 7 — Org-Wide Repo Scan
Quick scan of all repos for new issues and PRs that need routing.

- gh repo list Molecule-AI --limit 50 --json name --jq '.[].name' | while read repo; do
    gh issue list --repo Molecule-AI/$repo --state open --limit 3 --json number,title,labels
    gh pr list --repo Molecule-AI/$repo --state open --limit 3 --json number,title,author
  done

- Flag any issue with no area: label
- Flag any PR waiting > 4h without Security/QA review requested
- Route new issues to appropriate Lead

**Action:** PM updates sprint-backlog.md with any newly routed issues.

## STEP 8 — Response Body Sampling
Spot-check A2A responses to catch hallucination or quality drift early.

- Sample 3-5 recent A2A responses from different agents
- Verify: did the agent cite actual tool outputs (exact file paths, exact error messages)?
- Verify: did the agent make any unverifiable claims about infrastructure?
- Verify: did the agent follow the 4 philosophies in SHARED_RULES.md?

**Action:** If quality drift or hallucination found: notify the agent's Lead for coaching.

## Completion

After all 8 steps: commit_memory with one-line summary:
"maint-cycle HH:MM — output:OK token:OK unpushed:N review-routing:OK containers:N crons:OK org-scan:OK response:N"

If any step found issues: commit_memory notes the issues and the actions taken.
