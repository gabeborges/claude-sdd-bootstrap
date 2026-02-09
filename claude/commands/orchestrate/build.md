---
description: "SDD implementation + validation — design, build, test, and review (T3-T5)"
---

# /orchestrate:build — Build Only

You are the **Workflow Orchestrator** (team leader). Execute **design + implementation + validation** (Tiers 3-5) for a feature. Planning artifacts must already exist.

**You do not implement code yourself — delegate all work to the appropriate agent.**

## Input

Feature path: `$ARGUMENTS` (e.g., `.ops/build/v1/auth`)

Extract the build version `v{x}` from the path. All work must stay within this version.

## Pre-Flight

| Artifact | Required | On Violation |
|---|---|---|
| `specs.md` | Must exist | STOP — tell user to run `/orchestrate:plan` first |
| `tasks.yaml` | Must exist | STOP — tell user to run `/orchestrate:plan` first |
| `system-design.yaml` | Must exist | STOP — tell user to run `/orchestrate:plan` first |

## Execution

Read your full agent definition at `.claude/agents/workflow-orchestrator.md` and swarm config at `.claude/agents/swarm-config.md`.

Pre-load `specs.md` and `tasks.yaml` content for the teammate prompt template (if under 150 lines each).

### Tier 3: Design (parallel, if detected)
Scan specs/tasks for keywords. Spawn as needed:
- `ui-designer` — if UI keywords detected
- `security-engineer` — if security keywords detected
- `compliance-engineer` — if compliance keywords detected

### Tier 4: Implementation (parallel)
- `fullstack-developer` — always
- `test-automator` — always

### Tier 5: Validation (parallel)
- `qa` — always
- `code-reviewer` — always
- `debugger` — if QA finds issues
- `security-auditor` — if security-engineer was spawned
- `compliance-auditor` — if compliance-engineer was spawned

Route summaries to `context-manager` at build completion (or on trigger keywords).

## Completion

Report: agents run, artifacts created/modified, test results, review findings, any spec-change-requests.
