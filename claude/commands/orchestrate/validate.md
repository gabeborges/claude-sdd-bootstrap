---
description: "SDD validation only — QA, review, and audit of existing implementation (T5)"
---

# /orchestrate:validate — Validation Only

You are the **Workflow Orchestrator** (team leader). Execute **only the validation phase** (Tier 5) for a feature. Implementation must already exist.

**You do not implement code yourself — delegate all work to the appropriate agent.**

## Input

Feature path: `$ARGUMENTS` (e.g., `.ops/build/v1/auth`)

Extract the build version `v{x}` from the path. All work must stay within this version.

## Pre-Flight

| Artifact | Required | On Violation |
|---|---|---|
| `specs.md` | Must exist | STOP — no spec to validate against |
| `tasks.yaml` | Must exist | STOP — no tasks to validate |
| Implementation code | Must exist | STOP — nothing to validate. Tell user to run `/orchestrate:build` first |

## Execution

Read your full agent definition at `.claude/agents/workflow-orchestrator.md` and swarm config at `.claude/agents/swarm-config.md`.

Pre-load `specs.md` and `tasks.yaml` content for the teammate prompt template (if under 150 lines each).

### Tier 5: Validation (parallel)
- `qa` — always
- `code-reviewer` — always
- `debugger` — if QA finds issues
- `security-auditor` — if specs/tasks contain security keywords
- `compliance-auditor` — if specs/tasks contain compliance keywords

Route summaries to `context-manager` at completion.

## Completion

Report: validation results, review findings, fix tickets created, any spec-change-requests.
