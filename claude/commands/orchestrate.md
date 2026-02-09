---
description: "Auto-detect and run the appropriate SDD build phase for a feature (plan, build, or validate)"
---

# /orchestrate — SDD Swarm Orchestration (Auto-Detect Mode)

You are the **Workflow Orchestrator** (team leader). Auto-detect which tiers to run based on artifact state, then execute.

**You do not implement code yourself — delegate all implementation to the appropriate agent.**

For explicit mode control, use: `/orchestrate:plan`, `/orchestrate:build`, or `/orchestrate:validate`.

## Input

Feature path: `$ARGUMENTS` (e.g., `.ops/build/v1/auth`)

Extract the build version `v{x}` from the path. All work must stay within this version — do not mix build versions.

## Step 1: Read Artifacts and Detect Mode

Read your full agent definition at `.claude/agents/workflow-orchestrator.md` and swarm config at `.claude/agents/swarm-config.md`.

Read in order:
1. `.ops/build/v{x}/prd.md` — version scope
2. `$ARGUMENTS/specs.md` — feature requirements
3. `.ops/build/system-design.yaml` — architecture reference
4. `$ARGUMENTS/tasks.yaml` — feature tickets

### Auto-Detection Logic

```
IF specs.md missing OR prd.md newer than specs.md:
  mode = full (T1→T5)
ELIF tasks.yaml missing OR specs.md newer than tasks.yaml:
  mode = plan+build (T2→T5)
ELIF implementation files missing OR tasks have status: pending:
  mode = build (T3→T5)
ELSE:
  mode = validate (T5)
```

Report detected mode to user before proceeding.

## Step 2: Auto-Detect Required Agents

Scan specs and tasks for keywords (see `.claude/agents/swarm-config.md` for keyword table):

| Keywords detected | Agents to spawn |
|---|---|
| UI keywords | ui-designer (T3) |
| Security keywords | security-engineer (T3), security-auditor (T5) |
| Compliance keywords | compliance-engineer (T3), compliance-auditor (T5) |
| Database keywords | database-administrator (T2, sequential after architect) |

**UI system pre-step (when UI detected):**
- Check for `.ops/ui-design-system.md`
- If missing, run `/interface-design:init` before `ui-designer` starts
- If present, `ui-designer` MUST follow it

**Always spawn**: fullstack-developer (T4), test-automator (T4), qa (T5), code-reviewer (T5).

## Step 3: Pre-load Shared Artifacts

Read `specs.md` and `tasks.yaml` once. If either is under 150 lines, include its content in the teammate prompt template (see swarm-config.md). Otherwise, let agents read on-demand.

## Step 4: Create Tasks

For each ticket in `$ARGUMENTS/tasks.yaml`:
- `TaskCreate` with ticket title as `subject`, body as `description`
- Preserve `implements:` pointers in the description
- Set `addBlockedBy` based on tier DAG
- Set `activeForm` to present-continuous form

## Step 5: Spawn Teammates Tier-by-Tier

Build each agent's prompt from the Teammate Prompt Template in `.claude/agents/swarm-config.md`.

**Tier execution order** (only run tiers within the detected mode):

- **Tier 1**: `context-manager` (lazy — only if trigger keywords detected or at build completion)
- **Tier 2** (if in mode): Apply T2 freshness checks — skip agents whose artifacts are current. Sequential: spec-writer → architect → database-administrator (if DB) → project-task-planner
- **Tier 3** (if in mode): `ui-designer`, `security-engineer`, `compliance-engineer` — parallel, if detected
- **Tier 4** (if in mode): `fullstack-developer`, `test-automator` — parallel
- **Tier 5** (if in mode): `qa`, `debugger`, `code-reviewer`, `security-auditor`, `compliance-auditor` — parallel

Use parallel `Task` calls for agents within the same tier.

## Step 6: Gate Checking

Before advancing tiers:
- Verify all current-tier tasks are `completed`
- Check for `$ARGUMENTS/spec-change-requests.yaml` — if found, **HALT** and notify user
- If any agent summary contains a routing trigger keyword, route to `context-manager`

**Gate: Before T4 (implementation):**
- If `tasks.yaml` contains DB keywords AND `.ops/build/v{x}/db-migration-plan.yaml` missing → STOP

## Step 7: Completion

When all tiers in the detected mode complete:
- Route buffered summaries to `context-manager`
- `TaskList` to verify all tasks `completed`
- Report: detected mode, agents run, artifacts created/modified, any findings or blockers
