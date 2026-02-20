---
name: sdd-protocols
description: SDD protocol reference — spec-change-requests, checks.yaml, artifact chain, escalation rules
---

# SDD Protocols

Reference for SDD workflow protocols. Load this when you need to understand artifact prerequisites, escalation decision trees, or gate output formats.

---

## 1. Artifact Prerequisite Chain

Canonical order: `prd.md` → `specs.md` → `system-design.yaml` → `db-migration-plan.yaml` (conditional) → `tasks.yaml`

| Artifact | Prerequisite | On Violation |
|---|---|---|
| `specs.md` | `prd.md` must exist for that version | STOP — ask user |
| `system-design.yaml` | `specs.md` must exist for that feature | STOP — run spec-writer first |
| `db-migration-plan.yaml` | `system-design.yaml` complete AND specs contain DB keywords | STOP — run architect first |
| `tasks.yaml` | Both `specs.md` AND `system-design.yaml` must exist | STOP — run architect first |

**Rules:**
- Do not create `tasks.yaml` until `system-design.yaml` exists
- Do not create `system-design.yaml` until `specs.md` exists
- `specs.md` is authoritative — code must align with it, not the other way around
- All `implements:` pointers in `tasks.yaml` must reference nodes defined in `specs.md`

---

## 2. spec-change-requests.yaml

### Location
`.ops/build/v{x}/<feature-name>/spec-change-requests.yaml`

### Purpose
Formal mechanism to escalate spec gaps, ambiguities, or conflicts discovered during design/implementation/audit.

### When to Create
- **Ambiguous or missing information** prevents correct acceptance criteria (spec-writer)
- **Architecture conflicts with spec** — design patterns required differ from spec assumptions (architect)
- **Implementation reveals spec gap** — code review or QA finds spec divergence (code-reviewer, qa)
- **Security/compliance gap** — required controls not defined in spec (security-auditor, compliance-auditor)
- **Test requirements conflict with spec** (test-automator)

### Who Creates
- spec-writer (ambiguity during spec authoring)
- architect (system-design conflicts)
- qa (contract/scenario mismatch)
- code-reviewer (PR divergence from spec)
- security-auditor (missing security requirements)
- compliance-auditor (missing compliance requirements)
- test-automator (acceptance criteria ambiguity)
- database-administrator (schema change breaks existing queries)

### Format
```yaml
# .ops/build/v{x}/<feature-name>/spec-change-requests.yaml
requests:
  - id: SCR-001
    created_by: <agent-name>
    section: "<specs.md section or node path>"
    issue: "Description of gap/ambiguity/conflict"
    impact: "What is blocked or at risk"
    proposed_resolution: "Suggested fix (optional)"
    status: open|addressed|rejected
```

### What Happens After
1. Creator STOPS work immediately
2. workflow-orchestrator detects file, halts swarm
3. User or spec-writer addresses requests
4. spec-writer updates `specs.md`
5. Downstream artifacts regenerated if needed (system-design.yaml, tasks.yaml)
6. Work resumes

### Stop Condition
If `spec-change-requests.yaml` exists with `status: open`, STOP all implementation work until resolved.

---

## 3. checks.yaml (Merge-Only Protocol)

### Location
`.ops/build/v{x}/<feature-name>/checks.yaml`

### Purpose
Single YAML file for all gate results (security, compliance, QA, code review, testing, DB migration).

### Merge-Only Protocol
- **DO NOT overwrite** the entire file
- **DO write/update** only your agent's section
- Use YAML merge semantics or read-modify-write
- Preserve all other sections

### Section Schemas

#### db_migration
**Written by**: database-administrator

```yaml
db_migration:
  status: pass|fail
  blockers: []
  notes: []
```

**Example:**
```yaml
db_migration:
  status: pass
  blockers: []
  notes:
    - "Expand/contract pattern applied"
    - "~50k rows, backfill < 5s"
```

---

#### qa_validation
**Written by**: qa

```yaml
qa_validation:
  status: pass|fail
  blockers: []
  notes: []
  evidence: []
```

**Example:**
```yaml
qa_validation:
  status: fail
  blockers:
    - "Pagination: nextCursor is null instead of absent (T-004 created)"
  notes:
    - "Contract check: GET /users response matches UserList schema"
  evidence:
    - "src/routes/users.ts"
```

---

#### testing
**Written by**: test-automator

```yaml
testing:
  status: pass|fail
  blockers: []
  notes: []
  evidence: []
```

**Example:**
```yaml
testing:
  status: pass
  blockers: []
  notes:
    - "All acceptance criteria from specs.md covered"
    - "Contract tests validate response shapes"
  evidence:
    - "tests/routes/users.test.ts"
```

---

#### code_review
**Written by**: code-reviewer

```yaml
code_review:
  status: pass|fail
  blockers: []
  notes: []
  evidence: []
```

**Example:**
```yaml
code_review:
  status: fail
  blockers:
    - "Rate limiting middleware not applied (required by security.md)"
  notes:
    - "Response matches UserList schema"
    - "Cursor pagination works"
  evidence:
    - "src/routes/users.ts"
```

---

#### security_audit
**Written by**: security-auditor

```yaml
security_audit:
  status: pass|fail
  blockers: []
  notes: []
  evidence: []
```

**Example:**
```yaml
security_audit:
  status: fail
  blockers:
    - "SEV-2: Missing input length validation on user.name (src/routes/users.ts:23)"
  notes:
    - "express@4.18.2 clean, jsonwebtoken@9.0.0 clean"
  evidence:
    - "src/routes/users.ts"
```

**Findings Template (for notes/blockers):**
```markdown
SEV-1|SEV-2|SEV-3: {finding title}
- Location: {file:line}
- Issue: {description}
- Impact: {what could go wrong}
- Remediation: {required fix}
- Ticket: T-{NNN}
```

---

#### compliance_audit
**Written by**: compliance-auditor

```yaml
compliance_audit:
  status: pass|fail
  blockers: []
  notes: []
  evidence: []
```

**Example:**
```yaml
compliance_audit:
  status: fail
  blockers:
    - "CRITICAL: PHI logged in plaintext (src/routes/patients.ts:67)"
  notes:
    - "PHI encryption at rest: met"
    - "Audit log on access: met"
    - "Access control on /patients: met"
  evidence:
    - "src/routes/patients.ts"
    - "compliance.md 1.3"
```

**Findings Template (for notes/blockers):**
```markdown
CRITICAL|HIGH|MEDIUM: {finding title}
- Requirement: {compliance.md reference}
- Location: {file:line}
- Issue: {description}
- Remediation: {required fix}
- Ticket: T-{NNN}
```

---

### Status Values
- `pass` — gate satisfied, no blockers
- `fail` — gate not satisfied, blockers exist

### Blocker Protocol
If `status: fail`, agent MUST:
1. Create remediation tickets in `tasks.yaml` with clear repro/fix steps
2. Reference ticket IDs in `blockers` list

---

## 4. Escalation Decision Tree

### When to STOP (Immediate Halt)
- **Prerequisite missing**: artifact chain violated (see section 1)
- **Ambiguous spec**: information needed to write correct acceptance criteria
- **Security conflict**: spec contradicts security.md or OWASP patterns
- **Compliance conflict**: spec contradicts compliance.md requirements
- **Destructive DB operation**: no rollback plan or backfill verification
- **spec-change-requests.yaml exists** with `status: open`

**Action**: STOP work, notify user or workflow-orchestrator

---

### When to Create spec-change-requests.yaml
- Spec gap discovered during:
  - Architecture design (architect)
  - Implementation (via code-reviewer or qa)
  - Security/compliance audit
  - Test authoring (test-automator)
- Schema change breaks existing queries (database-administrator)
- Implementation is correct but spec is wrong

**Action**: Create entry in `spec-change-requests.yaml`, STOP work

---

### When to Proceed (No Escalation)
- Spec is clear and complete
- No conflicts with security.md, compliance.md, or system-design.yaml
- DB changes follow expand/contract pattern with rollback
- All gates pass (`checks.yaml` sections show `status: pass`)
- No open `spec-change-requests.yaml` entries

**Action**: Continue to next tier or mark task complete

---

## 5. Gate Output Protocol

### checks.yaml Merge Semantics
1. Read existing `checks.yaml` (if present)
2. Update only your section
3. Write back without overwriting other sections
4. If file doesn't exist, create it with only your section

### Gate Pass Criteria
- `status: pass`
- `blockers: []`
- `notes` may contain observations (non-blocking)

### Gate Fail Criteria
- `status: fail`
- `blockers` contains at least one blocking issue
- Remediation tickets created in `tasks.yaml`

---

## 6. implements: Pointer Protocol

### Purpose
Traceable connection from task to spec contract.

### Format
In `tasks.yaml`:
```yaml
- id: T-001
  title: "Implement GET /users endpoint"
  implements: /paths/users/get
  status: pending
```

### Rules
- `implements:` value MUST reference a node defined in `specs.md`
- Node format: `/paths/users/get`, `/components/UserList`, `/scenarios/S1`
- QA validates the `implements:` pointer — response shape must match spec contract
- Code reviewer verifies PR changes satisfy the `implements:` pointer

### Validation
- test-automator: map `implements:` pointer to test cases
- qa: verify implementation matches spec contract at that pointer
- code-reviewer: ensure PR diff aligns with `implements:` pointer's requirements

---

## 7. Remediation Ticket Protocol

### When to Create
- qa: contract mismatch, scenario failure
- security-auditor: vulnerability found
- compliance-auditor: compliance gap found
- code-reviewer: PR requires changes before merge
- debugger: root cause identified

### Format
```yaml
- id: T-{NNN}
  title: "Fix: {description}"
  found_by: <agent-name>
  blocks: T-{original-task-id}
  repro: "{exact steps to reproduce}"
  expected: "{what the spec says}"
  actual: "{what the implementation does}"
  status: pending
```

### Flow
1. Gate agent finds issue
2. Create remediation ticket in `tasks.yaml`
3. Mark original task as `blocked_by: T-{NNN}`
4. Update `checks.yaml` with `status: fail`, reference ticket in `blockers`
5. debugger or fullstack-developer picks up remediation ticket
6. Once fixed, re-run gate

---

## 8. Quick Reference

### File Locations
- Specs: `.ops/build/v{x}/<feature-name>/specs.md`
- System Design: `.ops/build/system-design.yaml`
- Tasks: `.ops/build/v{x}/<feature-name>/tasks.yaml`
- Gates: `.ops/build/v{x}/<feature-name>/checks.yaml`
- Escalations: `.ops/build/v{x}/<feature-name>/spec-change-requests.yaml`
- DB Plans: `.ops/build/v{x}/<feature-name>/db-migration-plan.yaml`

### Agent Flow
1. spec-writer → `specs.md`
2. architect → `system-design.yaml`
3. database-administrator → `db-migration-plan.yaml` (conditional — if specs contain DB keywords)
4. project-task-planner → `tasks.yaml`
5. Conditional agents (ui-designer, security-engineer, compliance-engineer)
6. frontend-designer (if ui-designer was spawned)
7. fullstack-developer + test-automator → code + tests
8. qa + code-reviewer + security-auditor + compliance-auditor → `checks.yaml`

### Stop Signals
- `spec-change-requests.yaml` exists with `status: open`
- `checks.yaml` has any section with `status: fail`
- Prerequisite artifact missing

### Resume Signals
- All `spec-change-requests.yaml` entries resolved
- All `checks.yaml` sections show `status: pass`
- All prerequisites exist
