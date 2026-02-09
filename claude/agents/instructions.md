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

Canonical flow: spec → system design → db migration planning (if DB changes) → tasks → safe execution → validation → gated release → logged state → spec feedback if needed.

### Definitions
- `specs.md` = feature requirements + acceptance criteria in `.ops/build/v{x}/<feature-name>/specs.md`
- `system-design.yaml` = architecture reference spanning multiple product versions in `.ops/build/system-design.yaml`
- `tasks.yaml` = feature tickets in `.ops/build/v{x}/<feature-name>/tasks.yaml`
- `implementation-status.md` = build-level progress tracker in `.ops/build/v{x}/implementation-status.md`

### SDD Artifact Flow (Prerequisite Rules)

Canonical order: `specs.md` (spec-writer) → `system-design.yaml` (architect) → `db-migration-plan.yaml` (database-administrator, build-level, if DB changes) → `tasks.yaml` (project-task-planner, per feature) → `build-order.yaml` (project-task-planner, build-level, if multi-feature)

| Artifact | Prerequisite | On Violation |
|---|---|---|
| `specs.md` | `prd.md` must exist for that version | STOP — ask user |
| `system-design.yaml` | `specs.md` must exist for that feature | STOP — run spec-writer first |
| `db-migration-plan.yaml` | `system-design.yaml` complete AND any feature's specs contain DB keywords (build-level, not per-feature) | STOP — run architect first |
| `tasks.yaml` | Both `specs.md` AND `system-design.yaml` must exist | STOP — run architect first |
| `build-order.yaml` | All features' `tasks.yaml` must exist | STOP — run project-task-planner for remaining features |

---

## Swarm Orchestration

The `/orchestrate` slash commands execute the SDD build phase using a swarm of agents managed by the `workflow-orchestrator` as team leader.

### Execution Modes

| Command | Tiers | Use Case |
|---------|-------|----------|
| `/orchestrate:plan` | T1-T2 | Planning only — specs, design, tasks |
| `/orchestrate:build` | T3-T5 | Design + implement + validate (requires existing specs/tasks) |
| `/orchestrate:validate` | T5 | QA + review only (requires existing implementation) |
| `/orchestrate` | Auto-detect | Routes to appropriate tier range based on artifact state |

### How It Works

1. **Entry**: `/orchestrate .ops/build/v{x}/<feature-name>` reads artifacts, detects mode (or uses explicit mode from slash command variant)
2. **Auto-detection**: Task and spec content is scanned for keywords to determine which optional agents to spawn (see `.claude/agents/swarm-config.md` for the full keyword table)
3. **Artifact pre-loading**: Orchestrator reads `specs.md` and `tasks.yaml` once, includes content in agent spawn prompts (if under 150 lines)
4. **Task creation**: Each ticket in `tasks.yaml` becomes a `TaskCreate` entry with `implements:` pointers preserved and `addBlockedBy` set per the tier DAG
5. **Tier-based spawning**: Agents spawn via the `Task` tool as `general-purpose` subagents, tier by tier within the mode's range
6. **Gate checking**: Each tier must complete before the next starts. If any agent creates `spec-change-requests.yaml`, the orchestrator halts and notifies the user
7. **Summary routing**: Event-driven — context-manager invoked on trigger keywords or at build completion (not per-tier)
8. **Completion**: All tasks verified complete, results summarized

### Agent-to-Teammate Mapping

Each agent's system prompt is read from `.claude/agents/<agent-name>.md` and combined with the feature workspace path to form the teammate's prompt. See `.claude/agents/swarm-config.md` for the full roster, tiers, and auto-detection triggers.

### Message Protocol

- Teammates return their summary to the orchestrator when done
- Spec violations → `spec-change-requests.yaml` → orchestrator halts
- Context-manager is the sole writer to `.ops/build/decisions-log.md` and `.ops/build/v{x}/implementation-status.md`
- Summary routing is event-driven: context-manager invoked on trigger keywords, after T2 checkpoint, and at build completion (not per-tier)
- `implements:` pointers flow from `tasks.yaml` into `TaskCreate` descriptions for traceability

---

## Dependency DAG

Per AGENTS.md — 5-tier structure:

```
Tier 1: context-manager  (lazy — invoked on triggers or at end)
   |
Tier 2: spec-writer → architect → database-administrator (if DB keywords) → project-task-planner  (sequential)
   |
Tier 3: ui-designer, security-engineer, compliance-engineer  (parallel, if detected)
   |
Tier 4: fullstack-developer, test-automator  (parallel)
   |
Tier 5: qa, debugger, code-reviewer, security-auditor, compliance-auditor  (parallel)
```

**Always spawn**: fullstack-developer (T4), test-automator (T4), qa (T5), code-reviewer (T5).

---

## Agent Inputs and Outputs

The canonical agent roster lives in **AGENTS.md**. This table adds Inputs/Outputs detail per agent.

| Agent | Role | Inputs (reads) | Outputs (writes) |
|---|---|---|---|
| workflow-orchestrator | Orchestration + routing | `specs.md`, `tasks.yaml`, `.ops/build/decisions-log.md`, repo status | Updates `tasks.yaml` ordering (optional); creates `spec-change-requests.yaml` when needed |
| context-manager | Decision log + deviation logger (sole writer) | Agent summaries, build artifacts, spec-change-requests | Maintains append-only `decisions-log.md` and `implementation-status.md` |
| spec-writer | Spec authoring + feature breakdown | `prd.md`, product context | `specs.md` (feature requirements + acceptance criteria) |
| architect | System design maintainer | `specs.md`, existing `system-design.yaml` | `.ops/build/system-design.yaml`; creates `spec-change-requests.yaml` if spec/architecture mismatch found |
| project-task-planner | Spec handoff / ticket writer | `specs.md`, `system-design.yaml`, `db-migration-plan.yaml` (if present), existing `.ops/build/decisions-log.md` | `tasks.yaml` (tickets with `implements:` pointers) |
| ui-designer | UX flows + component architecture | `specs.md`, `tasks.yaml` scope, existing UI patterns and codebase components, `.ops/ui-design-system.md` | `.ops/build/v{x}/<feature>/ui.md` (flows, screens, states, component architecture); updates `specs.md` with UX acceptance checks |
| fullstack-developer | Primary builder | `tasks.yaml`, `specs.md`, UI specs, `db-migration-plan.yaml` (if any), repo code | Code changes + tests |
| database-administrator | Migration strategist / safety gate | `specs.md` schema intent, `system-design.yaml` (`data.entities`, `data.key_constraints`), current DB schema | `.ops/build/v{x}/db-migration-plan.yaml` (build-level, consolidated) |
| qa | Contract validation + exploratory | `specs.md` (`implements:` pointers), `tasks.yaml`, running app/test outputs | Validation results; may open issues in `tasks.yaml` |
| test-automator | Automated test implementer | `specs.md`, `tasks.yaml`, repo test setup | Test files + fixtures |
| debugger | Root-cause investigator | Failing test logs, QA repro steps, recent diffs | Fix tickets in `tasks.yaml` |
| code-reviewer | Quality gate | PR diff, `tasks.yaml`, `specs.md` | Review notes; may add required-fix tasks to `tasks.yaml` |
| security-engineer | Secure-by-design implementer | `tasks.yaml`, auth model, infra context | `.ops/build/v{x}/<feature>/security.yaml`; updates `specs.md` with security checks |
| security-auditor | Independent security reviewer | PR diff, runtime config, dependency list | Findings list; remediation tasks in `tasks.yaml` |
| compliance-engineer | Compliance-by-design | `tasks.yaml`, `specs.md`, data flows/PHI assumptions | `.ops/build/v{x}/<feature>/compliance.yaml`; updates `specs.md` with compliance checks |
| compliance-auditor | Independent compliance reviewer | PR diff, logs/audit trails, data handling | Findings list; remediation tasks in `tasks.yaml` |

---

## Folder Structure

Per AGENTS.md:

```
.ops/
├── product-vision-strategy.md          (canonical — do not load unless cross-domain)
├── quick-product-vision-strategy.md    (distilled: §1–§7, §12 — product agents)
├── security-compliance-baseline.md     (distilled: §10 partial, §11, §12, §15, §16 — security/compliance agents)
├── tech-architecture-baseline.md       (distilled: §8–§10, §12–§16 — architecture agents)
├── ui-design-system.md (optional)
└── build/
    ├── decisions-log.md
    ├── system-design.yaml
    └── v{x}/
        ├── prd.md
        ├── implementation-status.md
        ├── db-migration-plan.yaml
        ├── build-order.yaml (conditional — multi-feature builds)
        └── <feature-name>/
            ├── specs.md
            ├── tasks.yaml
            ├── ui.md (conditional — features with UI)
            ├── security.yaml (conditional — security-sensitive features)
            ├── compliance.yaml (conditional — compliance-sensitive features)
            ├── checks.yaml
            └── spec-change-requests.yaml (optional)
```

### Gate Output
- `.ops/build/v{x}/<feature-name>/checks.yaml`: single YAML file with sections for security/compliance/qa/code_review/testing.

### System Design (Product-Level, Evolving)
- `.ops/build/system-design.yaml`: AI-first system design for the whole product.
  - Updated after specs are generated for a build (`spec-writer` → `architect`).
  - If `architect` adds `spec-change-requests.yaml`, rerun `spec-writer` for impacted features.
