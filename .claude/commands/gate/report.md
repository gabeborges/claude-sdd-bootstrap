---
name: gate:report
description: Unified gate status report — aggregates all checks.yaml sections for a feature
---

# Gate Report

Aggregates all gate results from `checks.yaml` for a feature workspace. Shows which gates passed, which failed, and whether the feature is ready to merge.

---

## Usage

```
/gate:report .ops/build/v1/user-auth
```

**Arguments**: Path to feature workspace (e.g., `.ops/build/v{x}/<feature-name>`)

---

## Required Reading

Before running this command, understand the gate schema:

1. `.claude/skills/sdd-protocols/SKILL.md` — checks.yaml schema, gate sections, status values
2. `claude/agents/instructions.md` — SDD tier structure, validation gates

---

## checks.yaml Schema

Based on `.claude/skills/sdd-protocols/SKILL.md`, `checks.yaml` has these sections:

| Section | Written By | Schema |
|---------|-----------|--------|
| `db_migration` | database-administrator | `status`, `blockers`, `notes` |
| `qa_validation` | qa | `status`, `blockers`, `notes`, `evidence` |
| `testing` | test-automator | `status`, `blockers`, `notes`, `evidence` |
| `code_review` | code-reviewer | `status`, `blockers`, `notes`, `evidence` |
| `security_audit` | security-auditor | `status`, `blockers`, `notes`, `evidence` |
| `compliance_audit` | compliance-auditor | `status`, `blockers`, `notes`, `evidence` |

**Status values**: `pass` | `fail`

---

## Report Steps

### Step 1: Read checks.yaml

1. Parse `$ARGUMENTS` to get feature workspace path: `.ops/build/v{x}/<feature-name>`
2. Read `checks.yaml` from `.ops/build/v{x}/<feature-name>/checks.yaml`
3. If file doesn't exist, report: "No gate results found. Gates have not run yet."

### Step 2: Parse Gate Sections

For each gate section in `checks.yaml`:

1. Extract `status` (pass/fail)
2. Extract `blockers` (list of blocking issues)
3. Extract `notes` (non-blocking observations)
4. Extract `evidence` (file references, if present)

### Step 3: Aggregate Results

Count:
- Total gates (how many sections exist)
- Passed gates (`status: pass`)
- Failed gates (`status: fail`)
- Missing gates (expected section not present)

### Step 4: Determine Overall Verdict

**Merge Ready**: All gates present and `status: pass`
**Blocked**: One or more gates have `status: fail`
**Incomplete**: One or more expected gates missing

---

## Output Format

```markdown
# Gate Status Report: <feature-name>

**Workspace**: `.ops/build/v{x}/<feature-name>`
**Overall Status**: MERGE READY | BLOCKED | INCOMPLETE
**Gates Passed**: 4/6
**Gates Failed**: 2/6
**Gates Missing**: 0/6

---

## Gate Results

### ✅ DB Migration
**Status**: PASS
**Blockers**: None
**Notes**:
- Expand/contract pattern applied
- ~50k rows, backfill < 5s

---

### ✅ Testing
**Status**: PASS
**Blockers**: None
**Notes**:
- All acceptance criteria from specs.md covered
- Contract tests validate response shapes
**Evidence**:
- `tests/routes/users.test.ts`

---

### ❌ QA Validation
**Status**: FAIL
**Blockers**:
1. Pagination: nextCursor is null instead of absent (T-004 created)
**Notes**:
- Contract check: GET /users response matches UserList schema
**Evidence**:
- `src/routes/users.ts`

---

### ✅ Code Review
**Status**: PASS
**Blockers**: None
**Notes**:
- Response matches UserList schema
- Cursor pagination works
**Evidence**:
- `src/routes/users.ts`

---

### ❌ Security Audit
**Status**: FAIL
**Blockers**:
1. SEV-2: Missing input length validation on user.name (src/routes/users.ts:23)
**Notes**:
- express@4.18.2 clean, jsonwebtoken@9.0.0 clean
**Evidence**:
- `src/routes/users.ts`

---

### ✅ Compliance Audit
**Status**: PASS
**Blockers**: None
**Notes**:
- PHI encryption at rest: met
- Audit log on access: met
- Access control on /patients: met
**Evidence**:
- `src/routes/patients.ts`
- `compliance.md 1.3`

---

## Summary

### Gates Passed (4)
- ✅ db_migration
- ✅ testing
- ✅ code_review
- ✅ compliance_audit

### Gates Failed (2)
- ❌ qa_validation
- ❌ security_audit

### Gates Missing (0)
- None

---

## Blockers List

**Total Blockers**: 2

1. **QA Validation** (qa)
   - Pagination: nextCursor is null instead of absent (T-004 created)

2. **Security Audit** (security-auditor)
   - SEV-2: Missing input length validation on user.name (src/routes/users.ts:23)

---

## Overall Verdict

**Status**: BLOCKED

**Why**: 2 gates have failing status with blockers

**What's Needed**:
1. Fix blocker: T-004 (QA validation — pagination response)
2. Fix blocker: SEV-2 input validation (Security audit — user.name validation)
3. Re-run qa and security-auditor agents to verify fixes
4. Once all gates pass, feature is merge ready

---

## Next Steps

**Immediate Actions**:
1. Address T-004: Fix pagination response in `src/routes/users.ts`
2. Address SEV-2: Add input length validation for user.name in `src/routes/users.ts`

**After Fixes**:
1. Re-run `/gate:report` to verify all gates pass
2. If all pass, proceed with merge
```

---

## Special Cases

### No checks.yaml
If `checks.yaml` doesn't exist, report:
```
Status: INCOMPLETE
Reason: No gate results found
Next Steps: Run validation agents (Tier 6) to generate gate results
```

### Partial checks.yaml
If only some sections exist (e.g., only `qa_validation` and `testing`), report:
```
WARNING: Incomplete gate coverage
Missing gates: db_migration, code_review, security_audit, compliance_audit
Recommendation: Run all Tier 6 agents before merging
```

### All Gates Pass
If all sections have `status: pass`, report:
```
Status: MERGE READY ✅

All validation gates passed. Feature is ready to merge.

Final Checklist:
- [ ] All blockers resolved
- [ ] All gates re-run and passing
- [ ] No open spec change requests
- [ ] Implementation status updated
```

---

## Gate Expectations

Expected gates (based on `.claude/skills/sdd-protocols/SKILL.md`):

1. **db_migration** — required if feature touches database schema
2. **qa_validation** — always required (contract validation)
3. **testing** — always required (test coverage)
4. **code_review** — always required (quality gate)
5. **security_audit** — always required (vulnerability scan)
6. **compliance_audit** — required if feature handles PHI/PII

**Missing Gates Detection**:
- If `db-migration-plan.yaml` exists but `db_migration` section missing → report as MISSING
- If compliance.md exists but `compliance_audit` section missing → report as MISSING

---

## Agent Protocol

**Mode**: Read-only aggregation

**What You DO:**
- Read `checks.yaml`
- Parse all gate sections
- Aggregate results
- Determine overall verdict
- Report blockers

**What You DON'T DO:**
- Modify `checks.yaml`
- Run validation gates
- Fix blockers
- Update implementation status

---

## Verification Checklist

At the end of gate report, confirm:

- [ ] Read `checks.yaml` successfully
- [ ] Parsed all gate sections
- [ ] Counted passed/failed/missing gates
- [ ] Listed all blockers
- [ ] Determined overall verdict
- [ ] Provided actionable next steps
