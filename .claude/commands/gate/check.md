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

### Step 3: Check for Blockers

Check for blocking conditions:

1. **spec-change-requests.yaml** — `.ops/build/v{x}/<feature-name>/spec-change-requests.yaml`
   - If exists with `status: open`, BLOCKED
   - List open request IDs

2. **checks.yaml** — `.ops/build/v{x}/<feature-name>/checks.yaml`
   - If exists, check each section for `status: fail`
   - List failing gates

### Step 4: Determine Readiness Tier

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

**Tier 4 Ready** (Implementation prep)
- All Tier 2 artifacts exist
- Tier 3 outputs present (if agents were triggered)
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
- [ ] Checked for blockers (spec-change-requests, failing gates)
- [ ] Determined current tier
- [ ] Identified next tier blockers
- [ ] Generated readiness report
