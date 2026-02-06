---
name: security:review
description: Security review of code changes — checks auth, OWASP, input validation, secrets, RLS
---

## Required Reading — Do This First

Before starting the review, read these files completely:

1. `~/.claude/skills/security-patterns/SKILL.md` — security patterns, auth, validation, secrets
2. `~/.claude/skills/security-patterns/references/owasp-checklist.md` — detailed OWASP Top 10 checks

Do not skip this. The security knowledge is in these files.

---

## Scope

Security review for code changes in Next.js + Supabase + Stripe stack.

**Use for:**
- Pre-merge code reviews
- Security audits of new features
- Compliance validation
- Post-incident analysis

**Not for:**
- Infrastructure security (hosting, network)
- Performance optimization
- General code quality (use code-reviewer for that)

---

## Arguments

`$ARGUMENTS` can be:
- **File path** (e.g., `app/api/users/route.ts`)
- **Directory** (e.g., `app/api/`)
- **PR reference** (e.g., `#123` or PR URL)
- **Commit range** (e.g., `main...feature-branch`)

If no arguments provided, review all staged changes.

---

## Review Process

### 1. Load Context

Read the security-patterns skill knowledge:
- Authentication patterns (Supabase Auth, sessions, RLS)
- Input validation patterns (Zod schemas)
- Secrets management rules
- OWASP Top 10 checks

### 2. Identify Changes

Determine what to review:
- If file/directory: Read those files
- If PR: Fetch PR diff using `gh pr view` or `gh pr diff`
- If commit range: Use `git diff`
- If no args: Use `git diff --staged`

### 3. Security Scan

Walk through each check category:

#### **Authentication & Authorization**
- [ ] All endpoints require authentication (unless explicitly public)
- [ ] User ID extracted from `session.user.id`, NEVER from request
- [ ] Supabase RLS enabled on all tables with user data
- [ ] RLS policies match authorization requirements
- [ ] No client-side auth logic (auth checks on server)
- [ ] Admin routes verify role from session
- [ ] No service role key exposed to client

#### **Input Validation**
- [ ] All server endpoints validate input (Zod schemas or equivalent)
- [ ] Length limits on string inputs (prevent DoS)
- [ ] Type validation (string, number, boolean, enum)
- [ ] Format validation (email, URL, UUID)
- [ ] Enum whitelisting for fixed value sets
- [ ] Sanitization of HTML/Markdown if displayed
- [ ] Parameterized queries only (no SQL string concatenation)

#### **Secrets Management**
- [ ] No hardcoded secrets (API keys, passwords, tokens)
- [ ] All secrets in environment variables
- [ ] No secrets in logs or console.log statements
- [ ] No `NEXT_PUBLIC_*` variables for sensitive data
- [ ] `.env` files in `.gitignore`
- [ ] No secrets in staged changes (check diffs)

#### **Supabase RLS**
- [ ] RLS enabled on all user data tables
- [ ] Policies use `auth.uid()` for user identification
- [ ] No RLS bypass on client (only server with service role, sparingly)
- [ ] RLS policies tested with multiple user accounts

#### **OWASP Top 10 Quick Scan**

Reference `~/.claude/skills/security-patterns/references/owasp-checklist.md` for detailed checks.

**A1: Broken Access Control**
- Auth checks present on endpoints
- Direct object references validated

**A2: Cryptographic Failures**
- No secrets in code/logs
- HTTPS enforced

**A3: Injection**
- Input validated
- Parameterized queries

**A4: Insecure Design**
- Threat model exists for new features
- Defense in depth

**A5: Security Misconfiguration**
- Security headers configured
- No default credentials

**A6: Vulnerable Components**
- Dependencies up to date
- No high/critical `npm audit` findings

**A7: Authentication Failures**
- Session verified on every request
- OAuth scopes minimal

**A8: Integrity Failures**
- Stripe webhook signatures verified

**A9: Logging Failures**
- Sensitive actions logged
- No PII/PHI/secrets in logs

**A10: SSRF**
- User-provided URLs validated/whitelisted

#### **Stripe Security** (if applicable)
- [ ] Webhook signatures verified (`stripe.webhooks.constructEvent`)
- [ ] Price IDs determined server-side (not from client)
- [ ] No billing logic bypass

#### **Google OAuth** (if applicable)
- [ ] OAuth scopes not expanded without justification
- [ ] Callback URLs whitelisted

### 4. Output Findings Report

Format findings by severity level:

```markdown
## Security Review: {path/PR/commit}

### Summary
- Total issues: {count}
- SEV-1 (Critical): {count}
- SEV-2 (High): {count}
- SEV-3 (Medium): {count}

### SEV-1: Critical Issues (Block Merge)

**Finding: {Description}**
- **Location**: `{file}:{line}`
- **Risk**: {What could go wrong}
- **Evidence**: {Code snippet or pattern}
- **Remediation**: {Specific fix}
- **Reference**: {OWASP category or skill doc section}

### SEV-2: High Issues (Block Merge)

{Same format as SEV-1}

### SEV-3: Medium Issues (Create Ticket)

{Same format as SEV-1}

### Recommendation

- **SEV-1/2 present**: REQUEST_CHANGES — Fix critical/high issues before merge
- **Only SEV-3**: APPROVE with comment — Create tickets for medium issues
- **No issues**: APPROVE — Security checks passed
```

### 5. Suggest Remediation

For each finding, provide:
- Exact file and line number
- Code snippet showing the issue
- Specific fix (with code example if applicable)
- Link to relevant section in `security-patterns/SKILL.md` or `owasp-checklist.md`

---

## Severity Levels

**SEV-1 (Critical) — BLOCK MERGE:**
- Secrets in code/logs/diffs
- Missing auth checks on privileged endpoints
- RLS disabled on user data tables
- SQL injection vulnerabilities
- Service role key exposed to client

**SEV-2 (High) — BLOCK MERGE:**
- Missing input validation
- Unverified Stripe webhook signatures
- Missing rate limiting on auth endpoints
- PII/PHI in logs
- User ID from request instead of session

**SEV-3 (Medium) — CREATE TICKET:**
- Missing security headers
- Outdated dependencies with known CVEs
- Missing audit logs
- Weak error messages (info leakage)
- No rate limiting on non-auth endpoints

---

## Special Cases

### New Feature Review

If reviewing a new feature, verify:
1. Threat model exists (or create one using STRIDE framework)
2. Security requirements defined
3. RLS policies documented
4. Input validation schemas present
5. Sensitive data handling documented

### Compliance Conflicts

If security requirements conflict with spec:
- **Stop the review**
- Create entry in `spec-change-requests.yaml`
- Document conflict: security requirement vs. spec requirement
- Escalate to architect/spec-writer

### Missing Context

If you can't assess security without more info:
- **Ask specific questions** (don't guess)
- Request threat model or security requirements
- Suggest running `/security-patterns:audit` for deeper analysis

---

## Example Output

```markdown
## Security Review: app/api/users/route.ts

### Summary
- Total issues: 3
- SEV-1 (Critical): 1
- SEV-2 (High): 1
- SEV-3 (Medium): 1

### SEV-1: Critical Issues

**Finding: User ID extracted from request query parameter**
- **Location**: `app/api/users/route.ts:12`
- **Risk**: Attacker can access any user's data by changing `userId` query param
- **Evidence**:
  ```typescript
  const userId = searchParams.get('userId') // Client controls this
  const { data } = await supabase.from('users').select('*').eq('id', userId)
  ```
- **Remediation**: Extract user ID from session instead:
  ```typescript
  const { data: { session } } = await supabase.auth.getSession()
  if (!session) return new Response('Unauthorized', { status: 401 })
  const userId = session.user.id
  ```
- **Reference**: `security-patterns/SKILL.md` — Authentication & Authorization, `owasp-checklist.md` A1: Broken Access Control

### SEV-2: High Issues

**Finding: Missing input validation on user profile update**
- **Location**: `app/api/users/route.ts:25`
- **Risk**: Malformed input could cause errors or bypass business logic
- **Evidence**: Request body used directly without validation
  ```typescript
  const body = await request.json()
  await supabase.from('users').update(body).eq('id', userId)
  ```
- **Remediation**: Add Zod schema validation:
  ```typescript
  import { z } from 'zod'

  const UpdateUserSchema = z.object({
    name: z.string().min(1).max(100),
    bio: z.string().max(500).optional(),
  })

  const result = UpdateUserSchema.safeParse(body)
  if (!result.success) {
    return new Response(JSON.stringify({ error: result.error }), { status: 400 })
  }

  await supabase.from('users').update(result.data).eq('id', userId)
  ```
- **Reference**: `security-patterns/SKILL.md` — Input Validation

### SEV-3: Medium Issues

**Finding: No rate limiting on endpoint**
- **Location**: `app/api/users/route.ts` (entire file)
- **Risk**: Endpoint could be overwhelmed by repeated requests
- **Remediation**: Add rate limiting middleware (see `security-patterns/SKILL.md` — Rate Limiting)
- **Reference**: `owasp-checklist.md` A5: Security Misconfiguration

### Recommendation

**REQUEST_CHANGES** — Fix SEV-1 and SEV-2 issues before merge. Create ticket for SEV-3 (rate limiting).
```

---

## Post-Review Actions

After completing the review:

1. **If SEV-1 or SEV-2 found:**
   - Request changes on PR (or notify reviewer if direct review)
   - Create remediation tickets in `tasks.yaml` (one per finding)
   - Block merge until fixed

2. **If only SEV-3 found:**
   - Approve with comment noting medium issues
   - Create tickets for deferred fixes
   - Document in `.ops/build/decisions-log.md` if applicable

3. **If no issues:**
   - Approve
   - Note in review comment: "Security checks passed (auth, validation, secrets, RLS, OWASP)"

4. **If spec conflict:**
   - Create `spec-change-requests.yaml` entry
   - Do not proceed with review until conflict resolved

---

## Communication Style

- Be precise and actionable
- Reference specific lines and files
- Provide code examples for fixes
- Link to skill docs for context
- Use severity levels consistently
- No guessing — ask if context missing
