## General Guidance

Deterministic, artifact-driven SDD workflow:
- Specs are authoritative (`specs.md`), not code.
- Tasks are traceable via `implements:` pointers back to spec nodes.
- QA validates contracts, not just behavior.
- DB changes are gated by `database-administrator` (no unsafe migrations).
- Security and compliance are first-class, with both by-design and audit roles.
- `workflow-orchestrator` closes the loop, routing implementation constraints back to spec-writer/user instead of allowing silent drift.
- `context-manager` persists state so work is resumable, auditable, and non-chatty.
- `context-manager` is the sole writer to `decisions-log.md` and `implementation-status.md` — records decisions, state changes, and deviations. No other agent writes to these files.
- All `implements:` pointers in `.ops/build/v{x}/<feature-name>/tasks.yaml` must reference nodes defined in `.ops/build/v{x}/<feature-name>/specs.md`. Example: `implements: /paths/users/get`

Canonical flow: spec → system design → tasks → safe execution → validation → gated release → logged state → spec feedback if needed.

### Definitions
- `specs.md` = feature requirements + acceptance criteria in `.ops/build/v{x}/<feature-name>/specs.md`
- `system-design.yaml` = architecture reference spanning multiple product versions in `.ops/build/system-design.yaml`
- `tasks.yaml` = feature tickets in `.ops/build/v{x}/<feature-name>/tasks.yaml`
- `implementation-status.md` = build-level progress tracker in `.ops/build/v{x}/implementation-status.md`

### SDD Artifact Flow (Prerequisite Rules)

Canonical order: `specs.md` (spec-writer) → `system-design.yaml` (architect) → `tasks.yaml` (project-task-planner)

| Artifact | Prerequisite | On Violation |
|---|---|---|
| `specs.md` | `prd.md` must exist for that version | STOP — ask user |
| `system-design.yaml` | `specs.md` must exist for that feature | STOP — run spec-writer first |
| `tasks.yaml` | Both `specs.md` AND `system-design.yaml` must exist | STOP — run architect first |

---

## Swarm Orchestration

The `/orchestrate <feature-path>` slash command executes the SDD build phase using a swarm of agents managed by the `workflow-orchestrator` as team leader.

### How It Works

1. **Entry**: `/orchestrate .ops/build/v{x}/<feature-name>` reads `specs.md` and `tasks.yaml` from the feature workspace, and uses build context from `../prd.md` and `.ops/build/system-design.yaml`
2. **Auto-detection**: Task and spec content is scanned for keywords to determine which optional agents to spawn (see `.claude/agents/swarm-config.md` for the full keyword table)
3. **Task creation**: Each ticket in `tasks.yaml` becomes a `TaskCreate` entry with `implements:` pointers preserved and `addBlockedBy` set per the tier DAG
4. **Tier-based spawning**: Agents spawn via the `Task` tool as `general-purpose` subagents, tier by tier (see Dependency DAG below)
5. **Gate checking**: Each tier must complete before the next starts. If any agent creates `spec-change-requests.yaml`, the orchestrator halts and notifies the user
6. **Completion**: All tasks verified complete, results summarized

### Agent-to-Teammate Mapping

Each agent's system prompt is read from `.claude/agents/<agent-name>.md` and combined with the feature workspace path to form the teammate's prompt. See `.claude/agents/swarm-config.md` for the full roster, tiers, and auto-detection triggers.

### Message Protocol

- Teammates return their summary to the orchestrator when done
- Spec violations → `spec-change-requests.yaml` → orchestrator halts
- Context-manager is the sole writer to `.ops/build/decisions-log.md` and `.ops/build/v{x}/implementation-status.md` (batch per-tier, keyword-triggered deviations)
- `implements:` pointers flow from `tasks.yaml` into `TaskCreate` descriptions for traceability

---

## Dependency DAG

Per AGENTS.md — 6-tier structure:

```
Tier 1: workflow-orchestrator, context-manager
   |
Tier 2: spec-writer → architect → project-task-planner  (sequential)
   |
Tier 3: ui-designer, security-engineer, compliance-engineer  (parallel, if detected)
   |
Tier 4: frontend-designer, database-administrator  (parallel, if detected)
   |
Tier 5: fullstack-developer, test-automator  (parallel)
   |
Tier 6: qa, debugger, code-reviewer, security-auditor, compliance-auditor  (parallel)
```

**Always spawn**: context-manager (T1), fullstack-developer (T5), test-automator (T5), qa (T6), code-reviewer (T6).

---

## Agent Inputs and Outputs

The canonical agent roster lives in **AGENTS.md**. This table adds Inputs/Outputs detail per agent.

| Agent | Role | Inputs (reads) | Outputs (writes) |
|---|---|---|---|
| workflow-orchestrator | Orchestration + routing | `specs.md`, `tasks.yaml`, `.ops/build/decisions-log.md`, repo status | Updates `tasks.yaml` ordering (optional); creates `spec-change-requests.yaml` when needed |
| context-manager | Decision log + deviation logger (sole writer) | Agent summaries, build artifacts, spec-change-requests | Maintains append-only `decisions-log.md` and `implementation-status.md` |
| spec-writer | Spec authoring + feature breakdown | `prd.md`, product context | `specs.md` (feature requirements + acceptance criteria) |
| architect | System design maintainer | `specs.md`, existing `system-design.yaml` | `.ops/build/system-design.yaml`; creates `spec-change-requests.yaml` if spec/architecture mismatch found |
| project-task-planner | Spec handoff / ticket writer | `specs.md`, `system-design.yaml`, existing `.ops/build/decisions-log.md` | `tasks.yaml` (tickets with `implements:` pointers) |
| ui-designer | UX intent + flows | `specs.md`, `tasks.yaml` scope, existing UI patterns, `.ops/ui-design-system.md` | UX flows, screens, states, accessibility notes |
| frontend-designer | Design-to-implementation translator | UI specs, codebase components, `tasks.yaml` | Component breakdown (components/props/states) |
| fullstack-developer | Primary builder | `tasks.yaml`, `specs.md`, UI specs, `db-migration-plan.yaml` (if any), repo code | Code changes + tests |
| database-administrator | Migration strategist / safety gate | `specs.md` schema intent, current DB schema, `tasks.yaml` | `db-migration-plan.yaml` (expand/contract/backfill/rollback) |
| qa | Contract validation + exploratory | `specs.md` (`implements:` pointers), `tasks.yaml`, running app/test outputs | Validation results; may open issues in `tasks.yaml` |
| test-automator | Automated test implementer | `specs.md`, `tasks.yaml`, repo test setup | Test files + fixtures |
| debugger | Root-cause investigator | Failing test logs, QA repro steps, recent diffs | Fix tickets in `tasks.yaml` |
| code-reviewer | Quality gate | PR diff, `tasks.yaml`, `specs.md` | Review notes; may add required-fix tasks to `tasks.yaml` |
| security-engineer | Secure-by-design implementer | `tasks.yaml`, auth model, infra context | Security patterns/decisions; updates `specs.md` with security checks |
| security-auditor | Independent security reviewer | PR diff, runtime config, dependency list | Findings list; remediation tasks in `tasks.yaml` |
| compliance-engineer | Compliance-by-design | `tasks.yaml`, `specs.md`, data flows/PHI assumptions | Compliance requirements; updates `specs.md` with compliance checks |
| compliance-auditor | Independent compliance reviewer | PR diff, logs/audit trails, data handling | Findings list; remediation tasks in `tasks.yaml` |

---

## Folder Structure

Per AGENTS.md:

```
.ops/
├── product-vision-strategy.md
├── ui-design-system.md (optional)
└── build/
    ├── decisions-log.md
    ├── system-design.yaml
    └── v{x}/
        ├── prd.md
        ├── implementation-status.md
        └── <feature-name>/
            ├── specs.md
            ├── tasks.yaml
            └── (optional) checks.yaml, db-migration-plan.yaml, spec-change-requests.yaml
```

### Gate Output
- `.ops/build/v{x}/<feature-name>/checks.yaml`: single YAML file with sections for security/compliance/qa/code_review/testing.

### System Design (Product-Level, Evolving)
- `.ops/build/system-design.yaml`: AI-first system design for the whole product.
  - Updated after specs are generated for a build (`spec-writer` → `architect`).
  - If `architect` adds `spec-change-requests.yaml`, rerun `spec-writer` for impacted features.
