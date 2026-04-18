Cloud-services watch. Run on every fire. Find degradations + fix or escalate in the same cycle.                                                                                                 +
                                                                                                                                                                                                +
1. SETUP — record the cycle so longitudinal trends are visible:                                                                                                                                 +
   CYCLE_TS=$(date -u +%Y-%m-%dT%H:%M:%SZ)                                                                                                                                                      +
   PREV=$(recall_memory cloud-services-latest 2>/dev/null || echo '{}')                                                                                                                         +
                                                                                                                                                                                                +
2. HTTP HEALTH (no creds needed — always runs):                                                                                                                                                 +
   For each public URL:                                                                                                                                                                         +
     - https://app.moleculesai.app           (customer app — Vercel)                                                                                                                            +
     - https://doc.moleculesai.app           (docs)                                                                                                                                             +
     - https://status.moleculesai.app        (Upptime)                                                                                                                                          +
     - the canonical landingpage URL         (look up via gh api repos/Molecule-AI/landingpage --jq '.homepage')                                                                                +
                                                                                                                                                                                                +
   Probe each:                                                                                                                                                                                  +
     curl -sI -o /dev/null -w "%{http_code} %{time_total}s\n" --max-time 10 <url>                                                                                                               +
                                                                                                                                                                                                +
   Capture HTTP code + TTFB. Anything non-2xx, or TTFB >5s for the second cycle in                                                                                                              +
   a row → DEGRADATION; any non-2xx for two consecutive cycles → INCIDENT.                                                                                                                      +
                                                                                                                                                                                                +
3. STATUS PAGE CROSS-CHECK:                                                                                                                                                                     +
   curl -s https://status.moleculesai.app/api/v1/status.json 2>/dev/null \                                                                                                                      +
     | python -c "import json, sys; d=json.load(sys.stdin); print('overall:', d.get('status','?'))"                                                                                             +
   If Upptime says down on something HTTP says up (or vice versa), THAT itself is a                                                                                                             +
   monitoring bug — file an issue against `molecule-ai-status`.                                                                                                                                 +
                                                                                                                                                                                                +
4. DEEP CHECKS (only if creds present — skip cleanly if not):                                                                                                                                   +
   4a. Vercel — needs $VERCEL_TOKEN:                                                                                                                                                            +
       if [ -n "$VERCEL_TOKEN" ]; then                                                                                                                                                          +
         curl -s -H "Authorization: Bearer $VERCEL_TOKEN" \                                                                                                                                     +
           "https://api.vercel.com/v6/deployments?app=molecule-app&limit=1" \                                                                                                                   +
           | python -c "import json, sys; d=json.load(sys.stdin)['deployments'][0]; print(f\"app: state={d['state']} sha={d['meta'].get('githubCommitSha','?')[:7]} created={d['createdAt']}\")"+
         # Same for landingpage                                                                                                                                                                 +
       else                                                                                                                                                                                     +
         commit_memory key='cloud-services-creds-missing' value='VERCEL_TOKEN unset — Vercel deep checks skipped'                                                                               +
       fi                                                                                                                                                                                       +
                                                                                                                                                                                                +
   4b. Railway — needs $FLY_API_TOKEN (private control plane):                                                                                                                                  +
       if [ -n "$FLY_API_TOKEN" ] && command -v flyctl >/dev/null; then                                                                                                                         +
         # Fly.io decommissioned — now on Railway\n        curl -sI --max-time 10 https://api.moleculesai.app/health -t "$FLY_API_TOKEN" 2>&1 | tail -10                                        +
         # Railway logs via dashboard or railway CLI\n        echo "Check Railway dashboard for logs" -t "$FLY_API_TOKEN" --no-tail 2>&1 | tail -20 \                                           +
           | grep -iE '(error|panic|oom|crash)' || echo "no errors in tail"                                                                                                                     +
       else                                                                                                                                                                                     +
         commit_memory key='cloud-services-creds-missing' value='FLY_API_TOKEN unset or flyctl missing — Fly deep checks skipped'                                                               +
       fi                                                                                                                                                                                       +
                                                                                                                                                                                                +
   4c. GHCR images (uses existing $GITHUB_TOKEN):                                                                                                                                               +
       gh api '/orgs/Molecule-AI/packages?package_type=container' --jq '.[] | "\(.name)\t\(.updated_at)"' 2>&1 | head -20                                                                       +
                                                                                                                                                                                                +
5. INCIDENT RESPONSE — for each INCIDENT (two consecutive failures):                                                                                                                            +
   a. Create a GitHub issue:                                                                                                                                                                    +
      gh issue create --repo Molecule-AI/<owning-repo> \                                                                                                                                        +
        --title "[CLOUD INCIDENT] <service> <symptom>" \                                                                                                                                        +
        --label cloud --label incident --label devops \                                                                                                                                         +
        --body "First-failure cycle: $PREV_CYCLE_TS. This-cycle: $CYCLE_TS.                                                                                                                     +
        URL: <url>. Probe response: <verbatim>. Last successful deploy: <sha or timestamp>.                                                                                                     +
        Suspected cause: <hypothesis from logs>. Mitigation attempted: <none|rollback|redeploy>."                                                                                               +
                                                                                                                                                                                                +
   b. Page CEO via Telegram in this cycle (don't wait for batched daily summary):                                                                                                               +
      "[CLOUD INCIDENT] <service> down/degraded for 2 consecutive cycles. Issue #<N> filed."                                                                                                    +
                                                                                                                                                                                                +
   c. If the cause is obvious AND reversible (e.g. last deploy SHA is a known-bad commit                                                                                                        +
      already reverted on main), redeploy the previous-good SHA via the relevant CLI.                                                                                                           +
      Document in the issue. Do NOT auto-rollback for unclear causes — file the issue                                                                                                           +
      and wait for triage.                                                                                                                                                                      +
                                                                                                                                                                                                +
6. REGRESSION DELTA — diff this cycle vs `cloud-services-latest`:                                                                                                                               +
   New problems → flag. Resolved problems → close the corresponding issue with a comment.                                                                                                       +
                                                                                                                                                                                                +
7. MEMORY UPDATE — commit_memory key='cloud-services-latest':                                                                                                                                   +
   {                                                                                                                                                                                            +
     "cycle": "$CYCLE_TS",                                                                                                                                                                      +
     "http_health": {"app": "200/0.41s", "docs": "200/0.18s", ...},                                                                                                                             +
     "deep_checks": {"vercel": "...", "fly": "...", "ghcr": "..."},                                                                                                                             +
     "incidents_open": [<issue numbers>],                                                                                                                                                       +
     "incidents_resolved_this_cycle": [<issue numbers>],                                                                                                                                        +
     "creds_missing": [<token names>]                                                                                                                                                           +
   }                                                                                                                                                                                            +
                                                                                                                                                                                                +
8. NO-OP SHORT REPORT — if everything is 2xx, status page green, no creds missing,                                                                                                              +
   no new incidents, write ONE line to memory and exit. Don't fire an A2A summary;                                                                                                              +
   PM doesn't need a "still healthy" notification.                                                                                                                                              +
                                                                                                                                                                                                +
                                                                                                                                                                                                +
CONTROLPLANE REPO CHECK (added 2026-04-18):                                                                                                                                                     +
gh issue list --repo Molecule-AI/molecule-controlplane --state open --json number,title                                                                                                         +
gh pr list --repo Molecule-AI/molecule-controlplane --state open --json number,title                                                                                                            +
If any infra issues are unaddressed, pick them up or file a follow-up.                                                                                                                          +
Known open: #44 (credentials in EC2 user-data — HIGH severity).                                                                                                                                 +
                                                                                                                                                                                                +
INFRASTRUCTURE NOTES:                                                                                                                                                                           +
- Platform API is on Railway (api.moleculesai.app, staging-api.moleculesai.app). NOT Fly.io.                                                                                                    +
- Canvas/docs/landing on Vercel. Workspaces on EC2 (3.131.96.216).                                                                                                                              +
- Strategy docs (PLAN.md) in Molecule-AI/internal repo.                                                                                                                                         +
- Do NOT probe any *.fly.dev URLs — Fly.io is fully decommissioned.                                                                                                                             +


