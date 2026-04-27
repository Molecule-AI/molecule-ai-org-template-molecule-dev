IMPORTANT: Check Molecule-AI/internal repo for roadmap (PLAN.md), known issues, runbooks before starting work.

Deep security audit — runs every 12 hours. Rotate focus areas to cover the full surface within 2 weeks.

## Focus Areas (rotate each cycle)

### Cycle A — Credential & Auth Surface
- All GitHub PATs / tokens: check issue dates, flag >90 days old
- All API keys (Stripe, Anthropic, OpenAI, Vercel, Fly): check rotation dates
- Auth middleware coverage: every handler has auth check?
- Session token expiry: tokens expire within 24h?
- No plaintext credentials in git history

### Cycle B — Dependency & Supply Chain
- Go: go vuln check ./... — any HIGH/CRITICAL CVEs?
- Node: npm audit --audit-level=high — any HIGH/CRITICAL advisories?
- Python: bandit -r . -ll — any HIGH issues?
- Check .github/workflows/ for vulnerable actions (use SHA-pinned actions)
- Check Dockerfile base images for outdated base images

### Cycle C — Data Isolation & Multi-Tenancy
- Tenant isolation: all DB queries filter by tenant_id?
- Cross-tenant data access: can User A access User B's workspace?
- Row-level security: any raw SQL without tenant_id filter?
- Webhook data: no cross-tenant data in webhook payloads?

### Cycle D — Network & Infrastructure
- Exposed ports: only intended ports open on production hosts?
- CORS: no Access-Control-Allow-Origin: * in production?
- Rate limiting: all public endpoints have rate limits?
- DNS: no direct IP hardcoding, all CNAMEs valid?

### Cycle E — Secrets Management
- SECRETS_MATRIX.md: all active secrets listed?
- Encryption at rest: credentials in DB using AES-256-GCM?
- Secret rotation: any secrets >90 days without rotation?
- Vault/backups: are encrypted backups being made?

## Output

For each finding, file a GitHub issue tagged area:security with severity label.

Report to Dev Lead + PM with summary: security-audit HH:MM — N critical, N high, N medium, N low. Issues: #A #B #C
