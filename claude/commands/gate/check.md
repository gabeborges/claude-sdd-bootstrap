---
name: gate:check
description: Pre-flight readiness check — shows which SDD tier a feature workspace is ready for
---

# Gate Check

Pre-flight readiness check for a feature workspace. Determines which SDD tier the workspace is ready for based on artifact presence and completeness.

---

## Usage

```
/gate:check .ops/build/v1/user-auth
```

**Arguments**: Path to feature workspace (e.g., `.ops/build/v{x}/<feature-name>`)

---

## Required Reading

Before running this command, understand the SDD tier structure:

1. `.claude/skills/sdd-protocols/SKILL.md` — artifact chain, tier prerequisites
2. `claude/agents/instructions.md` — SDD artifact flow, dependency DAG

---

## Tier Prerequisites

Based on `.claude/skills/sdd-protocols/SKILL.md`:

| Tier | Required Artifacts | Agent(s) |
|------|-------------------|----------|
| **Tier 1** | None (orchestration starts) | workflow-orchestrator, context-manager |
| **Tier 2** | `prd.md`, `specs.md`, `system-design.yaml`, `tasks.yaml` | spec-writer → architect → project-task-planner |
| **Tier 3** | All Tier 2 artifacts | ui-designer, security-engineer, compliance-engineer |
| **Tier 4** | All Tier 2 artifacts + Tier 3 outputs (if applicable) | frontend-designer, database-administrator |
| **Tier 5** | All Tier 2 artifacts + Tier 3-4 outputs (if applicable) | fullstack-developer, test-automator |
| **Tier 6** | All Tier 2-5 artifacts + implementation complete | qa, debugger, code-reviewer, security-auditor, compliance-auditor |

---

## Gate Check Steps

### Step 1: Read Feature Workspace

1. Parse `$ARGUMENTS` to get feature workspace path: `.ops/build/v{x}/<feature-name>`
2. Extract version (`v{x}`) and feature name from path

### Step 2: Check Artifact Chain

Check for presence and staleness of each artifact:

**Artifact Checks:**

1. **prd.md** — `.ops/build/v{x}/prd.md`
   - Present? Yes/No
   - Stale? Check if modified date is older than all feature specs

2. **specs.md** — `.ops/build/v{x}/<feature-name>/specs.md`
   - Present? Yes/No
   - Complete? Has Goal, Requirements, Acceptance Criteria sections
   - Stale? Check for `spec-change-requests.yaml` with `status: open`

3. **system-design.yaml** — `.ops/build/system-design.yaml`
   - Present? Yes/No
   - Stale? Check if modified date is older than `specs.md`

4. **tasks.yaml** — `.ops/build/v{x}/<feature-name>/tasks.yaml`
   - Present? Yes/No
   - Complete? All tasks have `implements:` pointers
   - Stale? Check if modified date is older than `specs.md` or `system-design.yaml`

5. **ui.md** (Tier 3, conditional) — `.ops/build/v{x}/<feature-name>/ui.md`
   - Required if: `specs.md` or `tasks.yaml` contains UI auto-detection keywords (`ui`, `screen`, `flow`, `component`, `view`, `page`, `layout`, `modal`, `form`, `button`, `navigation`, `responsive`, `wireframe`)
   - Present? Yes/No/Not Required
   - If required but missing: report as INCOMPLETE for Tier 3

6. **security.yaml** (Tier 3, conditional) — `.ops/build/v{x}/<feature-name>/security.yaml`
   - Required if: `specs.md` or `tasks.yaml` contains security auto-detection keywords (`auth`, `secret`, `token`, `credential`, `oauth`, `jwt`, `session`, `permission`, `rbac`, `acl`, `encryption`, `api key`, `password`, `mfa`, `2fa`)
   - Present? Yes/No/Not Required
   - If required but missing: report as INCOMPLETE for Tier 3

7. **compliance.yaml** (Tier 3, conditional) — `.ops/build/v{x}/<feature-name>/compliance.yaml`
   - Required if: `specs.md` or `tasks.yaml` contains compliance auto-detection keywords (`phi`, `pii`, `hipaa`, `phipaa`, `pipeda`, `compliance`, `audit trail`, `data retention`, `baa`, `encryption at rest`, `de-identification`, `access log`, `consent`, `gdpr`)
   - Present? Yes/No/Not Required
   - If required but missing: report as INCOMPLETE for Tier 3

8. **db-migration-plan.yaml** (build-level) — `.ops/build/v{x}/db-migration-plan.yaml`
   - Note: This is a build-level artifact, NOT per-feature
   - Required if: `specs.md` or `tasks.yaml` contains database auto-detection keywords (`schema`, `migration`, `table`, `column`, `index`, `database`, `db`, `foreign key`, `sql`, `relation`, `constraint`, `seed`, `backfill`)
   - Present? Yes/No/Not Required

9. **build-order.yaml** (build-level) — `.ops/build/v{x}/build-order.yaml`
   - Note: This is a build-level artifact, NOT per-feature
   - Required if: build version `v{x}` contains multiple feature directories
   - Check: List feature directories under `.ops/build/v{x}/`; if more than one exists, `build-order.yaml` is required
   - Present? Yes/No/Not Required

### Step 3: Check for Blockers

Check for blocking conditions:

1. **spec-change-requests.yaml** — `.ops/build/v{x}/<feature-name>/spec-change-requests.yaml`
   - If exists with `status: open`, BLOCKED
   - List open request IDs

2. **checks.yaml** — `.ops/build/v{x}/<feature-name>/checks.yaml`
   - If exists, check each section for `status: fail`
   - List failing gates

### Step 4: Check for Orphan Files

Check the feature workspace for non-canonical files. The approved file list for a feature workspace is:

**Canonical files** (per-feature):
- `specs.md`
- `tasks.yaml`
- `ui.md`
- `security.yaml`
- `compliance.yaml`
- `checks.yaml`
- `spec-change-requests.yaml`

**Check**:
1. List all files in `.ops/build/v{x}/<feature-name>/`
2. Compare against the canonical list above
3. If any file exists that is NOT in the canonical list, emit a warning:
   ```
   WARNING: Non-canonical file(s) found in feature workspace:
   - <filename>
   Recommendation: Move to appropriate location or remove if not needed
   ```
4. Orphan files do NOT block tier progression — this is a warning only

### Step 5: Determine Readiness Tier

Based on artifact presence:

**Tier 1 Ready** (Always ready — orchestration can start)
- Workspace path exists

**Tier 2 Ready** (Spec/Design/Tasks phase)
- `prd.md` exists
- Ready for: spec-writer, architect, project-task-planner

**Tier 3 Ready** (Optional design agents)
- All Tier 2 artifacts exist
- `specs.md`, `system-design.yaml`, `tasks.yaml` all present
- Ready for: ui-designer, security-engineer, compliance-engineer
- Auto-detection: scan `specs.md` and `tasks.yaml` for keywords (see swarm-config.md)

**Tier 4 Ready** (Implementation prep)
- All Tier 2 artifacts exist
- Tier 3 outputs present (if agents were triggered):
  - `ui.md` present if UI keywords detected
  - `security.yaml` present if security keywords detected
  - `compliance.yaml` present if compliance keywords detected
- Build-level: `db-migration-plan.yaml` present at `.ops/build/v{x}/db-migration-plan.yaml` (if DB keywords detected)
- Build-level: `build-order.yaml` present at `.ops/build/v{x}/build-order.yaml` (if multiple features in build)
- Ready for: frontend-designer, database-administrator

**Tier 5 Ready** (Implementation)
- All Tier 2-4 artifacts exist
- No open `spec-change-requests.yaml`
- Ready for: fullstack-developer, test-automator

**Tier 6 Ready** (Validation gates)
- All Tier 2-5 artifacts exist
- Implementation tasks marked complete
- Ready for: qa, code-reviewer, security-auditor, compliance-auditor

**Merge Ready**
- All Tier 6 gates pass (`checks.yaml` all `status: pass`)
- No open blockers

---

## Output Format

```markdown
# Gate Check Report: <feature-name>

**Workspace**: `.ops/build/v{x}/<feature-name>`
**Current Tier**: Tier 5 (Implementation)
**Status**: READY | BLOCKED | INCOMPLETE

---

## Artifact Status

| Artifact | Status | Modified | Notes |
|----------|--------|----------|-------|
| `prd.md` | ✅ Present | 2026-02-01 | - |
| `specs.md` | ✅ Present | 2026-02-02 | Complete, all sections present |
| `system-design.yaml` | ✅ Present | 2026-02-02 | In sync with specs |
| `tasks.yaml` | ✅ Present | 2026-02-03 | 10 tasks, all have `implements:` pointers |
| `ui.md` | ✅ Present | 2026-02-03 | Required (UI keywords detected) |
| `security.yaml` | ✅ Present | 2026-02-03 | Required (security keywords detected) |
| `compliance.yaml` | ⬚ Not Required | - | No compliance keywords detected |
| `db-migration-plan.yaml` | ✅ Present (build-level) | 2026-02-02 | `.ops/build/v1/db-migration-plan.yaml` |
| `build-order.yaml` | ⬚ Not Required | - | Single feature in build |
| `checks.yaml` | ⚠️ Present | 2026-02-04 | 2 failing gates (see Blockers) |

---

## Tier Readiness

- [x] **Tier 1** — Orchestration (always ready)
- [x] **Tier 2** — Spec/Design/Tasks (artifacts present)
- [x] **Tier 3** — Optional design agents (artifacts present)
- [x] **Tier 4** — Implementation prep (artifacts present)
- [x] **Tier 5** — Implementation (artifacts present)
- [ ] **Tier 6** — Validation gates (BLOCKED: failing gates)
- [ ] **Merge Ready** (BLOCKED: failing gates)

**Current Tier**: Tier 5 (Implementation)
**Next Tier**: Tier 6 (Validation gates) — BLOCKED

---

## Blockers

**Blocking Issues (2):**
1. `checks.yaml` → `security_audit` → `status: fail`
   - Blocker: "SEV-2: Missing input length validation on user.name"
2. `checks.yaml` → `qa_validation` → `status: fail`
   - Blocker: "Pagination: nextCursor is null instead of absent (T-004 created)"

**Open Spec Change Requests**: None

---

## Next Steps

**What's Blocking Tier 6:**
- Fix 2 failing gate checks in `checks.yaml`
- Security blocker: T-010 (fix input validation)
- QA blocker: T-004 (fix pagination response)

**Recommendation:**
1. Resolve blockers: T-004, T-010
2. Re-run security-auditor and qa agents
3. Once all gates pass, proceed to Tier 6

---

## Summary

**Ready For**: Tier 5 (Implementation) — remediation tasks in progress
**Blocked From**: Tier 6 (Validation) — 2 failing gates
**Missing**: None (all artifacts present)
**Stale**: None (all artifacts in sync)
```

---

## Special Cases

### No Artifacts
If workspace is empty (no artifacts), report:
```
Status: READY for Tier 1 (Orchestration start)
Next: Run /orchestrate to begin SDD workflow
```

### Stale Artifacts
If `system-design.yaml` is older than `specs.md`, report:
```
WARNING: system-design.yaml is stale (older than specs.md)
Recommendation: Re-run architect to sync system design with spec changes
```

### Open Spec Change Requests
If `spec-change-requests.yaml` exists with `status: open`, report:
```
BLOCKED: Open spec change requests exist
Cannot proceed until requests are resolved by spec-writer
```

---

## Agent Protocol

**Mode**: Read-only check

**What You DO:**
- Read all artifacts in workspace
- Check file timestamps
- Parse YAML/markdown for completeness
- Determine tier readiness
- Report blockers

**What You DON'T DO:**
- Modify any files
- Create missing artifacts
- Fix validation errors
- Run validation gates

---

## Verification Checklist

At the end of gate check, confirm:

- [ ] Read all expected artifacts
- [ ] Ran auto-detection keyword scan on `specs.md` and `tasks.yaml` for Tier 3 requirements
- [ ] Checked Tier 3 conditional outputs (`ui.md`, `security.yaml`, `compliance.yaml`)
- [ ] Checked build-level artifacts (`db-migration-plan.yaml`, `build-order.yaml`)
- [ ] Checked for orphan files in feature workspace
- [ ] Checked for blockers (spec-change-requests, failing gates)
- [ ] Determined current tier
- [ ] Identified next tier blockers
- [ ] Generated readiness report
