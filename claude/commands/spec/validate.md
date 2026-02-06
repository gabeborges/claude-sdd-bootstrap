---
name: spec:validate
description: Validate a feature workspace for spec completeness and task traceability
---

# Spec Validation

Validates a feature workspace to ensure:
- `specs.md` exists and has required sections
- `tasks.yaml` exists (if expected) and all tasks have valid `implements:` pointers
- Every spec node is covered by at least one task
- Every `implements:` pointer references a real spec node

---

## Usage

```
/spec:validate .ops/build/v1/user-auth
```

**Arguments**: Path to feature workspace (e.g., `.ops/build/v{x}/<feature-name>`)

---

## Required Reading

Before running this command, understand the SDD artifact protocol:

1. `.claude/skills/sdd-protocols/SKILL.md` — artifact chain, `implements:` pointer protocol, spec node format
2. `claude/agents/instructions.md` — artifact flow rules (specs → system-design → tasks)

---

## Validation Steps

### Step 1: Check Prerequisites

Verify the artifact chain is intact:

1. Read the feature workspace path from `$ARGUMENTS`
2. Check that `.ops/build/v{x}/<feature-name>/specs.md` exists
3. Check that the corresponding `prd.md` exists at `.ops/build/v{x}/prd.md`
4. If `specs.md` is missing, report: "Spec file not found. Run spec-writer first."

### Step 2: Validate specs.md Structure

Read `specs.md` and verify it contains these sections:

- **Goal** or **Objective** section
- **Requirements** section (functional/non-functional requirements)
- **Acceptance Criteria** section

**Check for spec nodes:**
- Scan for node paths like `/paths/users/get`, `/components/UserList`, `/scenarios/S1`
- Build a list of all defined spec nodes

**Report:**
- PASS: All required sections present
- FAIL: List missing sections

### Step 3: Validate tasks.yaml (if exists)

Check if `tasks.yaml` exists at `.ops/build/v{x}/<feature-name>/tasks.yaml`.

**If tasks.yaml exists:**

1. Parse all tasks
2. For each task, check if it has an `implements:` field
3. If missing `implements:`, report as WARNING
4. If `implements:` is present, verify the pointer references a node in `specs.md`

**Report:**
- PASS: All tasks have valid `implements:` pointers
- FAIL: List tasks with missing or invalid `implements:` pointers

### Step 4: Check Spec Coverage

For each spec node found in `specs.md`:
- Check if at least one task in `tasks.yaml` implements that node
- If no task covers a spec node, report as WARNING (orphaned spec node)

**Report:**
- PASS: All spec nodes covered by tasks
- WARNING: List orphaned spec nodes

### Step 5: Check for Orphaned Pointers

For each `implements:` pointer in `tasks.yaml`:
- Verify the pointer references an existing spec node
- If pointer references a non-existent node, report as FAIL

**Report:**
- PASS: All pointers reference valid spec nodes
- FAIL: List tasks with invalid pointers

---

## Output Format

```markdown
# Spec Validation Report: <feature-name>

**Workspace**: `.ops/build/v{x}/<feature-name>`
**Status**: PASS | WARNINGS | FAIL

---

## Results

### Prerequisites
- [x] `specs.md` exists
- [x] `prd.md` exists
- [x] `tasks.yaml` exists

### Spec Structure
- [x] Goal section present
- [x] Requirements section present
- [x] Acceptance Criteria section present
- [x] Spec nodes defined: 5

### Task Traceability
- [x] All tasks have `implements:` pointers (10/10)
- [x] All `implements:` pointers are valid (10/10)

### Coverage
- [x] All spec nodes covered by tasks (5/5)
- [ ] WARNING: 1 orphaned spec node: `/scenarios/S3` (no task implements this)

---

## Summary

**What Passed:**
- Spec structure complete
- All tasks traceable to spec nodes
- All pointers valid

**What Failed:**
- None

**What's Missing:**
- Orphaned spec node: `/scenarios/S3` needs a task or should be removed

---

## Next Steps

1. Create task for `/scenarios/S3`, or
2. Remove unused spec node from `specs.md`
```

---

## Special Cases

### No tasks.yaml
If `tasks.yaml` doesn't exist, report:
```
WARNING: tasks.yaml not found. This is expected if project-task-planner hasn't run yet.
Spec validation can only check structure, not traceability.
```

### No specs.md
If `specs.md` doesn't exist, STOP immediately:
```
FAIL: specs.md not found in workspace.
Cannot validate. Run spec-writer first.
```

### spec-change-requests.yaml exists
If `.ops/build/v{x}/<feature-name>/spec-change-requests.yaml` exists with `status: open`, report:
```
BLOCKED: Open spec change requests exist.
Validation cannot proceed until requests are resolved.
```

---

## Agent Protocol

**Mode**: Validation (read-only)

**What You DO:**
- Read specs.md, tasks.yaml, and prd.md
- Parse YAML and markdown
- Compare pointers against spec nodes
- Report findings

**What You DON'T DO:**
- Modify any files
- Create missing artifacts
- Fix validation errors

---

## Verification Checklist

At the end of validation, confirm:

- [ ] Read `specs.md` successfully
- [ ] Parsed spec nodes
- [ ] Read `tasks.yaml` (if exists)
- [ ] Validated all `implements:` pointers
- [ ] Checked coverage (spec → tasks)
- [ ] Checked for orphaned pointers (tasks → spec)
- [ ] Generated validation report
