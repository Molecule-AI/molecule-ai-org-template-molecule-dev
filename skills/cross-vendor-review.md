# Cross-Vendor Review Skill

**When to invoke:** Any PR touching authentication, billing, schema migrations, data deletion, or that changes behavior across multiple vendors (GitHub, Stripe, Vercel, Fly, Anthropic, OpenAI, etc.).

**Required before Gate 4 (Security) APPROVED** for High/Critical blast-radius changes.

---

## What "Cross-Vendor" Means

Cross-vendor = the change touches external service APIs, credential handling, OAuth flows, webhooks, or multi-system orchestration. Examples:

- Stripe billing integration (new endpoint, price change, webhook handler)
- GitHub OAuth / App installation flow
- Anthropic/OpenAI API key rotation or credential migration
- Vercel/Fly deployment configuration changes
- Multi-system secret rotation (changes affect 2+ vendor credentials)
- Data deletion that must propagate to external services
- Schema migration that changes stored external-service references

---

## Pre-Flight Checklist

Before starting the review, gather:

```
gh pr view <N> --json title,body,files,url
gh pr diff <N>
```

Confirm the PR actually touches cross-vendor surface. If it doesn't, return `N/A — no cross-vendor surface`.

---

## Review Steps

### Step 1 — Map the Vendor Surface

For each file changed, identify which external vendor it touches:

```
# Find API calls to external services
grep -rn "stripe\|github.com\|anthropic\|openai\|vercel\|fly.io" \
  --include="*.go" --include="*.ts" --include="*.js" \
  | grep -v "_test.go" | grep -v "vendor"
```

Document: vendor name → changed files → type of interaction (API call, webhook, config, credential)

### Step 2 — Check Each Vendor's Contract

For each vendor surface:

**Stripe:**
- [ ] Webhook signature verification is present and correct (`stripe.webhook.ConstructEvent`)
- [ ] Idempotency keys used for POST/PUT requests
- [ ] Price/plan IDs are validated against known values
- [ ] Refund logic handles partial vs full
- [ ] No hardcoded Stripe keys (should use env vars / secrets matrix)

**GitHub:**
- [ ] GitHub App token refresh handled (tokens expire after 1h)
- [ ] No raw PATs in code (use App installation tokens)
- [ ] Rate limiting respected (check `X-RateLimit-Remaining` headers)
- [ ] Webhook secret verification present

**Anthropic/OpenAI:**
- [ ] API keys stored in secrets, not in code
- [ ] Token counting implemented for cost control
- [ ] Retry logic with exponential backoff for 429/5xx
- [ ] No streaming responses that could truncate

**Vercel/Fly:**
- [ ] Deployment tokens scoped to specific environments
- [ ] Rollback procedure documented for each change
- [ ] Health check endpoints tested after deploy
- [ ] No direct production database writes from deployment scripts

### Step 3 — Credential Blast Radius

If the change affects credential handling:

- [ ] New credential type added to SECRETS_MATRIX.md?
- [ ] Old credential type properly rotated and revoked?
- [ ] Credential stored encrypted at rest (not plaintext DB)?
- [ ] Credential audit log entries written on create/rotate/revoke?

### Step 4 — Data Deletion Propagation

If the change involves data deletion:

- [ ] All external vendor data deleted (Stripe customers, GitHub App data, etc.)
- [ ] Deletion order is correct (external services first, then internal)
- [ ] Rollback plan documented (can deleted data be re-fetched from vendor?)
- [ ] Deletion confirmed via vendor dashboard before closing issue

### Step 5 — Schema Migration Safety

If the change involves DB schema migration:

- [ ] Migration is backward-compatible (old code can run against new schema)
- [ ] Rollback migration written and tested
- [ ] Zero-downtime migration strategy (if applicable)
- [ ] External vendor references in DB validated post-migration

### Step 6 — Failure Mode Analysis

For each vendor interaction, write the failure mode:

| Vendor | Operation | Failure Mode | Detection | Rollback |
|--------|-----------|-------------|-----------|----------|
| Stripe | Create customer | Duplicate email | Stripe error | Retry with idempotency key |
| GitHub | App token refresh | 401 on stale token | Catch + re-auth | Token refresh daemon |
| Fly | Deploy | Old image still running | `fly status` check | `fly deploy --image DOCKER_IMAGE` |

---

## Output Format

Post on the PR:

```
### Cross-Vendor Review (security-auditor-agent)

**Vendors touched:** Stripe (billing), GitHub (OAuth)
**Files:** src/handlers/billing.go, src/middleware/auth.go

#### Vendor Surface Assessment
- [APPROVED/ISSUES] Stripe: webhook signature verified, idempotency keys present
- [ISSUES] GitHub: App token refresh not handled in auth.go:47 — will 401 after 1h

#### Credential Blast Radius
- [OK] New Stripe key scoped to test environment only
- [ACTION] Rotate production Stripe key after deploy (DevOps to do)

#### Data Deletion (if applicable)
- N/A

#### Failure Modes
| Vendor | Op | Failure | Detection | Rollback |
|--------|-----|---------|-----------|----------|
| Stripe | CreateCustomer | Duplicate | 409 Conflict | Retry |

**VERDICT:** [CLEAR] / [CONCERNS: ...] / [BLOCKED]

If BLOCKED: specific file:line and required fix.
```

---

## Severity Escalation

- **Clear:** No cross-vendor concerns. Proceed to Gate 4 APPROVED.
- **Concerns:** Documented concerns, engineer can fix in same PR. Lead decides if re-review needed.
- **Blocked:** Requires structural change before any gate can pass. Engineer fixes, re-invoke this skill.

---

## Pitfalls

1. **Don't assume the vendor SDK handles everything.** Stripe SDK doesn't auto-verify webhook signatures — that's on you.
2. **Idempotency keys are not optional** on payment operations. Missing keys = duplicate charges on retry.
3. **Token expiry is not handled by the vendor.** GitHub App tokens expire after 1h. Your code must refresh.
4. **Rollback ≠ undo.** Rolling back a Stripe refund doesn't un-refund the customer. Know what each vendor's rollback actually does.
5. **Test in sandbox/staging first.** Don't test webhook handlers against production Stripe events.
