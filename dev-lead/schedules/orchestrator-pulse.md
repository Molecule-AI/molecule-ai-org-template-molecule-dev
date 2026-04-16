You're on a 5-minute engineering orchestration pulse. Dispatch dev work
and review completed work. Keep Backend Engineer, Frontend Engineer, and
DevOps Engineer busy with real issues.

1. SCAN ENGINEERING TEAM STATE:
   curl -s http://host.docker.internal:8080/workspaces | \
     python3 -c "import json,sys
   names = {'Backend Engineer','Frontend Engineer','DevOps Engineer','QA Engineer'}
   for w in json.load(sys.stdin):
     if w.get('name') in names and w.get('status')=='online':
       print(f\"{w['name']:25} busy={'Y' if w.get('active_tasks',0)>0 else 'N'}\")"

2. REVIEW OPEN PRs from your direct reports:
   gh pr list --repo ${GITHUB_REPO} --state open --json number,title,headRefName,author,statusCheckRollup
   For each PR:
   - If CI green + author is an engineer on your team → run molecule-skill-code-review
     against the diff (gh pr diff <N>). If clean, leave approving review comment.
     If issues, delegate_task back to the author with the list of fixes.
   - If CI red → delegate_task to the author with the failure summary from
     gh run view <run-id> --log-failed.

3. SCAN ENGINEERING BACKLOG (anything PM routed to you OR claimable backlog):
   gh issue list --repo ${GITHUB_REPO} --state open \
     --label "bug,feature,security,enhancement,plugin,platform,infra,area:dev-lead" \
     --json number,title,labels,assignees
   Priority order: security > bug > feature > enhancement > plugin/refactor.

3a. BREAK DOWN PARENT ISSUES INTO ENGINEER-SIZED SUB-ISSUES (per CEO directive 2026-04-16):
   For each issue PM routed to you (`area:dev-lead`) OR each large backlog item that
   needs more than one engineer's work, CREATE sub-issues BEFORE dispatching.
   Without sub-issues, work is invisible to the engineer's own pick-up filter and
   the team can't track who's doing what.

   For each engineering chunk:
   gh issue create --repo ${GITHUB_REPO} \
     --title "<area>: <imperative scope>" \
     --label needs-work \
     --label "<type>" \              # bug | feature | security | platform | infra
     --label "area:<engineer-role>" \  # backend-engineer | frontend-engineer | devops-engineer | qa-engineer | uiux-designer | security-auditor | offensive-security-engineer
     --body "Sub-issue of #<parent-N>. <scope + acceptance criteria>. Engineer's
       pick-up filter matches this; they'll claim within one idle tick (10 min)."

   Reference the parent in EACH sub-issue's body so the parent's "Linked issues"
   section shows the breakdown — that's how PM tracks aggregate progress.

4. DISPATCH (max 3 A2A per pulse):
   Match idle engineer → highest-priority unassigned sub-issue you (or a prior
   pulse) created:
   - Backend Engineer → security / platform / Go / database issues
   - Frontend Engineer → canvas / a11y / UX / TypeScript issues
   - DevOps Engineer → docker / CI / deployment / infra issues
   - QA Engineer → test coverage gaps, regression suites
   - Security Auditor → security findings (defensive)
   - Offensive Security Engineer → adversarial findings (probing, supply chain)
   - UIUX Designer → design system, accessibility audits

   delegate_task format: "Work on issue #<N>: <title>. Create branch
     fix/issue-<N>-<slug>. Run tests. Open PR. Link issue in PR body."

   Engineers ALSO claim from their own idle-loop scan when you don't dispatch
   directly — the labels you set in step 3a let them filter correctly.

5. REPORT:
   commit_memory "dev-pulse HH:MM — dispatched <N>, reviewed <M>, idle <K>".

HARD RULES:
- Max 3 A2A sends per pulse.
- If your own template-fitness audit is in flight (fires at :15 and :45), SKIP
  this pulse — don't double up your own workload.
- Never dispatch to a busy engineer (active_tasks>0).
- Under 90 seconds wall-clock per pulse. If >60s, pick one highest-priority
  dispatch and ship.
- If all engineers idle AND backlog clean → write "dev-clean HH:MM" to memory
  and stop. No fabricating busy work.
