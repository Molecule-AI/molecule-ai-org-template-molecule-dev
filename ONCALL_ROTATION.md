# On-Call Rotation

Every agent in the team is on a rotating on-call schedule. On-call means you are the first responder for incidents in your domain during your shift.

---

## Severity Levels

| Level | Definition | Response Time | Example |
|-------|-----------|--------------|---------|
| **P0 — Critical** | Production down, data breach, all users affected | Immediate (<5 min) | Platform returns 5xx for all requests |
| **P1 — High** | Major feature broken, significant user impact | 15 min | A2A message delivery fails |
| **P2 — Medium** | Degraded performance, non-critical feature broken | 1 hour | Slow workspace provisioning |
| **P3 — Low** | Minor issue, workaround exists | Next business day | UI glitch in non-critical flow |

---

## On-Call Schedule

| Team | Primary On-Call | Secondary On-Call | Rotation Cadence |
|------|----------------|------------------|-----------------|
| **Platform (molecule-core)** | Dev Lead | Backend Engineer | Weekly |
| **Controlplane** | CP Lead | CP-BE | Weekly |
| **Workspace Runtime** | Infra Lead | Infra-Runtime-BE | Weekly |
| **CI/CD** | Infra Lead | DevOps Engineer | Weekly |
| **App & Docs** | App Lead | Frontend Engineer | Weekly |
| **Security** | Security Auditor | Offensive Security Engineer | Weekly |
| **Marketing** | Marketing Lead | Content Marketer | Weekly |

**On-call week starts Monday 00:00 UTC and ends Sunday 23:59 UTC.**

---

## On-Call Responsibilities

### When Your Shift Starts

1. Read the last 20 lines of `cron-learnings.jsonl` — check for any ongoing incidents
2. Check `internal/security/incident-log.md` for active incidents
3. Verify your monitoring dashboards are reachable
4. Post `[<role>-agent] ON-CALL: starting shift` to memory

### During Your Shift

1. **P0/P1 alerts:** Drop everything and respond within 5/15 minutes
2. **P2/P3 alerts:** Handle within SLA, but monitor
3. **Incidents:** Follow the incident runbook in `internal/runbooks/incident-response.md`
4. **Handoffs:** Brief the incoming on-call at shift end

### Incident Response Protocol

```
P0 (Critical):
1. Acknowledge alert immediately
2. Post "[<role>-agent] INCIDENT P0: <description> — investigating" to memory
3. Assess blast radius: is this user-facing? data-affecting? multi-tenant?
4. Execute rollback if recent deploy is the suspected cause
5. Notify PM via A2A: "P0 incident: <description>. Impact: <users affected>. Investigating."
6. Coordinate with secondary on-call if needed
7. Do NOT speculate on root cause publicly until confirmed
8. When resolved: document in internal/security/incident-log.md

P1 (High):
1. Acknowledge within 15 minutes
2. Post to memory: "[<role>-agent] INCIDENT P1: <description> — investigating"
3. Assess if P0 escalation needed
4. Notify PM via A2A if user impact confirmed
5. When resolved: document in incident log
```

### When Your Shift Ends

1. Brief incoming on-call via A2A: current state, open issues, anything to watch
2. Post `[<role>-agent] ON-CALL: ending shift — <summary of shift>`
3. Update `internal/security/incident-log.md` with any unresolved items

---

## Rotation Assignment

PM owns the rotation schedule. PM maintains `internal/runbooks/oncall-schedule.md` with the current week's assignments.

Engineers: check `internal/runbooks/oncall-schedule.md` to know your shift. If you can't cover your shift, arrange a swap with another engineer and notify PM.

---

## On-Call Escalation Path

```
On-call engineer can't resolve?
→ Escalate to team Lead (15 min no resolution on P0)
→ Lead escalates to PM (30 min no resolution on P0)
→ PM escalates to CEO (P0 affecting all users for 30+ min)
```

---

## What On-Call Is NOT

- **On-call is NOT for routine PR review.** Use normal triage cycles.
- **On-call is NOT for development work.** Continue sprint tasks when no incidents.
- **On-call is NOT 24/7 for individuals.** Rotate weekly. Human rest is required.
- **On-call does NOT replace the 8-step maintenance cycle.** These run in parallel.

---

## Compensation

On-call hours count as working hours. If on-call duty exceeds reasonable load, discuss with PM.
