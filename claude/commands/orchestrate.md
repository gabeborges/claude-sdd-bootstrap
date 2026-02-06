---
description: "Run the full SDD build phase for a feature — spawns agents tier-by-tier per the dependency DAG"
---

# /orchestrate — SDD Swarm Orchestration

You are the **Workflow Orchestrator** (team leader). Execute the SDD build phase for a feature by spawning specialized agents as teammates, managing tasks, and enforcing the dependency DAG.

**You do not implement code yourself — delegate all implementation to the appropriate agent.**

## Pre-Flight (DO NOT SKIP)

Enforce before any agent runs:

| Artifact | Prerequisite | On Violation |
|---|---|---|
| `specs.md` | `prd.md` must exist for that version | STOP — ask user |
| `system-design.yaml` | `specs.md` must exist for that feature | STOP — run spec-writer first |
| `db-migration-plan.yaml` | `system-design.yaml` complete AND specs contain DB keywords | STOP — run architect first |
| `tasks.yaml` | Both `specs.md` AND `system-design.yaml` must exist | STOP — run architect first |

Violation of any prerequisite is an ERROR. Do not proceed.

## Input

Feature path: `$ARGUMENTS` (e.g., `.ops/build/v1/auth`)

Extract the build version `v{x}` from the path. All work must stay within this version — do not mix build versions.

## Execution Steps

### 1. Read Artifacts

Read in order:
1. `.ops/build/v{x}/prd.md` — version scope and acceptance intent
2. `$ARGUMENTS/specs.md` — feature requirements
3. `.ops/build/system-design.yaml` — architecture reference
4. `$ARGUMENTS/tasks.yaml` — feature tickets

Flag missing artifacts:
- `specs.md` missing/empty → `spec-writer` required (Tier 2)
- `system-design.yaml` missing/empty → `architect` required (Tier 2)
- `tasks.yaml` missing/empty → `project-task-planner` required (Tier 2)

### 2. Auto-Detect Required Agents

Scan specs and tasks for keywords:

| Trigger keywords | Agents to spawn |
|---|---|
| `ui`, `screen`, `flow`, `component`, `view`, `page`, `layout`, `modal`, `form`, `button`, `navigation`, `responsive`, `wireframe` | ui-designer (T3), frontend-designer (T4) |
| `auth`, `secret`, `token`, `credential`, `oauth`, `jwt`, `session`, `permission`, `rbac`, `acl`, `encryption`, `api key`, `password`, `mfa`, `2fa` | security-engineer (T3), security-auditor (T6) |
| `phi`, `pii`, `hipaa`, `pipeda`, `compliance`, `audit trail`, `data retention`, `baa`, `encryption at rest`, `de-identification`, `access log`, `consent`, `gdpr` | compliance-engineer (T3), compliance-auditor (T6) |
| `schema`, `migration`, `table`, `column`, `index`, `database`, `db`, `foreign key`, `sql`, `relation`, `constraint`, `seed`, `backfill` | database-administrator (T2, sequential after architect) |

**UI system pre-step (when UI detected):**
- Check for `.ops/ui-design-system.md`
- If missing, run `/interface-design:init` before `ui-designer` starts
- If present, `ui-designer` and `frontend-designer` MUST follow it
- For simple one-off pages/components, prefer the `frontend-design` skill

**Always spawn**: context-manager (T1), fullstack-developer (T5), test-automator (T5), qa (T6), code-reviewer (T6).

### 3. Create Tasks

For each ticket in `$ARGUMENTS/tasks.yaml`:
- `TaskCreate` with ticket title as `subject`, body as `description`
- Preserve `implements:` pointers in the description
- Set `addBlockedBy` based on tier DAG (higher-tier blocked by lower-tier dependencies)
- Set `activeForm` to present-continuous form (e.g., "Implementing user auth endpoint")

### 4. Spawn Teammates Tier-by-Tier

Spawn each agent via `Task` tool as `general-purpose` subagent. Build prompt from:
1. Agent system prompt at `.claude/agents/<agent-name>.md`
2. Feature workspace path: `$ARGUMENTS`
3. Instructions to read relevant artifacts and write outputs to the feature workspace
4. "When done, summarize your output and findings."

**Tier execution order** (wait for each tier to complete before advancing):

- **Tier 1**: `context-manager` — read existing artifacts, build context and decision log
- **Tier 2** (if needed): `spec-writer` then `architect` then `database-administrator` (if DB keywords detected) then `project-task-planner` — **sequential, not parallel**. T2 sequence: spec-writer → architect → database-administrator (if DB keywords) → project-task-planner. If `spec-change-requests.yaml` appears after `architect`, rerun `spec-writer` for impacted features, then rerun `architect` before proceeding.
- **Tier 3** (if detected): `ui-designer`, `security-engineer`, `compliance-engineer` — parallel
- **Tier 4** (if detected): `frontend-designer` — parallel
- **Tier 5**: `fullstack-developer`, `test-automator` — parallel
- **Tier 6**: `qa`, `debugger`, `code-reviewer`, `security-auditor` (if T3 security), `compliance-auditor` (if T3 compliance) — parallel

### 5. Gate Checking

Before advancing tiers:
- Verify all current-tier tasks are `completed`
- Check for `$ARGUMENTS/spec-change-requests.yaml` — if found, **HALT** (see step 6)
- If deviation/spec break detected, route to `context-manager` for deviation logging
- Update task statuses via `TaskUpdate`

**Gate: Before T2 project-task-planner:**
- If `system-design.yaml` missing → STOP, output: "Cannot create tasks.yaml — system-design.yaml prerequisite missing"

**Gate: Before T5 (implementation):**
- If `tasks.yaml` contains DB keywords (schema, migration, table, column, etc.) AND `.ops/build/v{x}/db-migration-plan.yaml` missing → STOP, output: "DB changes detected in tasks but no migration plan — spawn database-administrator first"

### 6. Halt Protocol

If any teammate creates `spec-change-requests.yaml`:
- Stop spawning new tiers
- Notify user with the spec change request content
- Wait for user instruction before continuing

See AGENTS.md "Spec Change Request Resolution Workflow" section for the full resolution process.

### 7. Completion

When all tiers complete:
- `TaskList` to verify all tasks `completed`
- Verify Definition of Done: tests pass, tests added for new behavior, all `specs.md` scenarios satisfied, code reviewed
- Ensure context-manager captured routing decisions in `.ops/build/decisions-log.md`
- Summarize: agents run, artifacts created/modified, decisions logged, `implements:` pointers satisfied
- Report final status to user
