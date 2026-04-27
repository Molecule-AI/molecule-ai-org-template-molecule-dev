# Security Audit Skill

**Invoked by:** Security Auditor, Security Auditor 2, and any engineer self-auditing before filing a PR.

**When to invoke:**
- Every hour: run the hourly-security-review.md schedule
- Every 12h: run the full security-audit-every-12h.md schedule
- Before any PR touching auth, billing, secrets, middleware, or handlers: self-audit using this skill
- On demand: `skill_view("security-audit")` when uncertain

---

## Quick Security Checklist

Run this before filing any auth/billing/secrets PR:

```
# 1. SQL injection — parameterization check
grep -rn "fmt.Sprintf.*SELECT\|fmt.Sprintf.*INSERT\|fmt.Sprintf.*UPDATE" \
  --include="*.go" | grep -v "_test.go" | grep -v "db.Query\|db.Exec"

# 2. Secrets in logs — no token values printed
grep -rn "log.*token\|log.*secret\|log.*key\|fmt.Print.*password" \
  --include="*.go" --include="*.ts" --include="*.js" | grep -v "_test.go"

# 3. ConstantTimeCompare for secret comparison
grep -rn "==.*token\|!=.*secret\|bytes.Equal" --include="*.go" | grep -v "ConstantTimeCompare"

# 4. Auth middleware on all handlers
grep -rn "func.*Handler" src/handlers/ | head -20
# Then check each handler has auth check

# 5. Tenant isolation — no cross-tenant queries
grep -rn "SELECT.*FROM" --include="*.go" src/ | grep -v "tenant_id\|WHERE" | grep -v "_test"

# 6. YAML injection
grep -rn "fmt.Sprintf.*yaml\|yaml.Marshal\|yaml.Unmarshal" --include="*.go" | grep -v "_test"
# Check that user input is validated before YAML encoding

# 7. XSS — no innerHTML, no dangerouslySetInnerHTML without sanitization
grep -rn "innerHTML\|dangerouslySetInnerHTML" --include="*.tsx" --include="*.jsx" | grep -v "sanitize"

# 8. Command injection
grep -rn "exec.Command\|os/exec" --include="*.go" | grep -v "_test.go"
# Check shell=False or validated input

# 9. CORS — not wildcard in production
grep -rn "Access-Control-Allow-Origin" --include="*.go" --include="*.ts"
```

---

## Detailed Audit Procedure

### Step 1 — Map the Attack Surface

For the changed code, identify:
- All user-controllable inputs (URL params, body, headers, cookies)
- All external service calls (DB, API calls to vendors)
- All data flows from input → processing → storage → response

### Step 2 — Check Each Input

For each user input, verify:
- Input is validated (type, range, format)
- Input is sanitized before use in: SQL, HTML, JS, YAML, shell commands, file paths
- Input length is bounded (prevents DoS)

### Step 3 — Check Each Auth Boundary

For each access control decision:
- Is the user authenticated? (check session/token)
- Is the user authorized for this resource? (check tenant_id, user_id, role)
- Is the authorization check before the operation, not after?

### Step 4 — Check Secrets Handling

For each secret:
- Stored encrypted at rest (AES-256-GCM, not plaintext)
- Transmitted over TLS only
- Never in logs, error messages, or API responses
- Compared with `ConstantTimeCompare` or `subtle.ConstantTimeCompare`
- Rotated if exposed (file issue for rotation)

### Step 5 — Check Dependency Security

```
# Go: check for known CVEs
go vuln check ./...

# Node: check npm audit
cd /workspace/repos/<repo> && npm audit --audit-level=high

# Python: check bandit
bandit -r . -ll
```

---

## Finding Severity

| Severity | Definition | Action |
|----------|-----------|--------|
| **CRITICAL** | Remote code execution, data breach, auth bypass | Fix immediately, block PR |
| **HIGH** | SQL injection, XSS, IDOR, SSRF | Fix before merge, block PR |
| **MEDIUM** | Missing rate limiting, weak crypto | Fix before merge |
| **LOW** | Informational, best practice | Document in PR |

---

## Output Format

Post findings on the PR:

```
### Security Review

**Files audited:** src/handlers/auth.go, src/middleware/auth.go

| Finding | Severity | File:Line | Fix |
|---------|----------|-----------|-----|
| SQL injection in GetUser | CRITICAL | auth.go:47 | Use parameterized query |
| Missing auth on /admin | HIGH | auth.go:112 | Add auth middleware |

**VERDICT:** [APPROVED] / [CHANGES REQUESTED] / [BLOCKED]
```

---

## Self-Audit vs Auditor Review

**Self-audit:** Engineer runs this skill before filing a PR. Catches obvious issues.
**Auditor review:** Security Auditor independently reviews PR. Catches what engineer missed.

Both are required. Self-audit does NOT replace auditor review.
