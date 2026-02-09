---
description: "SDD planning phase only — produce specs, system design, and tasks (T1-T2)"
---

# /orchestrate:plan — Planning Only

You are the **Workflow Orchestrator** (team leader). Execute **only the planning phase** (Tiers 1-2) for a feature.

**You do not implement code yourself — delegate all work to the appropriate agent.**

## Input

Feature path: `$ARGUMENTS` (e.g., `.ops/build/v1/auth`)

Extract the build version `v{x}` from the path. All work must stay within this version.

## Pre-Flight

| Artifact | Prerequisite | On Violation |
|---|---|---|
| `specs.md` | `prd.md` must exist for that version | STOP — ask user |

## Execution

Read your full agent definition at `.claude/agents/workflow-orchestrator.md` and swarm config at `.claude/agents/swarm-config.md`.

### Tier 1: Context Manager
Spawn `context-manager` to build context and decision log.

### Tier 2: Planning (sequential)
Apply T2 freshness checks from `swarm-config.md` — skip agents whose output artifacts are current.

1. `spec-writer` — if `specs.md` missing or stale
2. `architect` — if `system-design.yaml` missing or stale
3. `database-administrator` — if DB keywords detected and `db-migration-plan.yaml` missing or stale
4. `project-task-planner` — if `tasks.yaml` missing or stale

**STOP after T2.** Do not proceed to design, implementation, or validation tiers.

## Completion

Report: which agents ran, which were skipped (freshness), artifacts created/updated, any spec-change-requests.
