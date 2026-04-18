Orchestrator check-in (every 2h). Light-touch coordination only — engineers drive their own work now.

STEP 1 — TEAM OUTPUT CHECK (do NOT delegate — just observe):
  gh pr list --repo Molecule-AI/molecule-core --state open --json number,title,author,createdAt
  Check: are engineers opening PRs? If no new PRs from app/molecule-ai in last 2h, note which engineers are idle.

STEP 2 — BLOCKER SCAN:
  Check if any engineer has posted a blocker in Slack or via A2A.
  Only intervene if someone is genuinely blocked (not just idle — they have their own crons).

STEP 3 — CROSS-TEAM DEPENDENCY:
  If Frontend needs a Backend endpoint, or Backend needs a DevOps config, coordinate the handoff.
  Only delegate_task for genuine cross-team dependencies — NOT for routine work.

STEP 4 — REPORT (brief):
  Who shipped what since last pulse. Who is blocked and on what.
  Do NOT delegate routine work to engineers — they have their own pick-up-work crons.

RULES:
- Engineers self-organize via hourly work crons. Your job is unblocking, not assigning.
- All PRs target staging. Merge-commits only.
- Do NOT delegate to PM unless there is a CEO-level decision needed.
