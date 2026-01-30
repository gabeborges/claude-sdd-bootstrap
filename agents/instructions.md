## General guidance
You finalized a deterministic, artifact-driven SDD workflow where:
* Specs are authoritative (OpenSpec), not code.
* Tasks are traceable via implements: pointers back to spec nodes.
* QA validates contracts, not just behavior.
* DB changes are gated by a Database Change Guardian (no unsafe migrations).
* Security and compliance are first-class, with both by-design and audit roles.
* Claude Tasks act as the execution workspace, while repo artifacts are the source of truth.
* "workflow-orchestrator" agent closes the loop, routing implementation constraints back to Clavix/OpenSpec instead of allowing silent drift.
* "context-manager" persists state so work is resumable, auditable, and non-chatty.
* Agent template: you should follow this guidance when creating agents: https://github.com/agentsmd/agents.md
* All implements: pointers in: `./ops/build/v{x}/<feature>/tasks.md` must reference nodes defined in: `./ops/build/v{x}/<feature>/spec.md`. Example: implements: `/paths/users/get`

In short: spec → tasks → safe execution → validation → gated release → logged state → spec feedback if needed.

## Swarm Orchestration

The `/orchestrate <feature-path>` slash command executes the SDD build phase using a swarm of agents managed by the `workflow-orchestrator` as team leader.

### How it works

1. **Entry**: `/orchestrate .ops/build/v1/<feature>` reads `spec.md` and `tasks.md` from the feature workspace
2. **Auto-detection**: Task and spec content is scanned for keywords to determine which optional agents to spawn (see `agents/swarm-config.md` for the full keyword table)
3. **Task creation**: Each ticket in `tasks.md` becomes a `TaskCreate` entry with `implements:` pointers preserved and `addBlockedBy` set per the tier DAG
4. **Tier-based spawning**: Agents spawn via the `Task` tool as `general-purpose` subagents, tier by tier:
   - T1: context-manager (always) → T2: project-task-planner (if needed) → T3: ui-designer, security-engineer, compliance-engineer (if detected, parallel) → T4: frontend-designer, database-administrator (if detected, parallel) → T5: fullstack-developer, test-automator (parallel) → T6: qa, code-reviewer, security-auditor, compliance-auditor, debugger (parallel)
5. **Gate checking**: Each tier must complete before the next starts. If any agent creates `spec-change-requests.md`, the orchestrator halts and notifies the user
6. **Completion**: All tasks verified complete, results summarized

### Agent-to-teammate mapping

Each agent's system prompt is read from `agents/<agent-name>.md` and combined with the feature workspace path to form the teammate's prompt. See `agents/swarm-config.md` for the full roster, tiers, and auto-detection triggers.

### Message protocol

- Teammates return their summary to the orchestrator when done
- Spec violations → `spec-change-requests.md` → orchestrator halts
- Context-manager appends all decisions to `decisions.md`
- `implements:` pointers flow from `tasks.md` into `TaskCreate` descriptions for traceability

## Agents team (final list)

| Subagent | Role | Inputs (reads) | Outputs (writes) | SDD workflow responsibility |
| --- | --- | --- | --- | --- |
| workflow-orchestrator | Orchestration + routing | `spec.md`, `tasks.md`, `acceptance.md`, `decisions.md`, repo status | Updates `tasks.md` ordering (optional), creates `spec-change-requests.md` when needed | Enforces the SDD sequence; triggers the right agents; routes spec breaks back to Clavix/OpenSpec update loop; ensures required gates ran |
| context-manager | Memory + decision log | Everything in `features/<feature>/` + PR/commit references | Appends to `decisions.md` (what changed/decisions/follow-ups) | Maintains a stable “execution state” in-repo so each iteration starts where you left off |
| project-task-planner | Spec handoff / ticket writer | `spec.md`, Clavix intent summary (if stored), existing `decisions.md` | `tasks.md` (tickets with `implements:` pointers), optional updates to `acceptance.md` skeleton | Parses OpenSpec and creates implementable tickets with `implements:` pointers + acceptance hooks |
| ui-designer | UX intent + flows | Clavix intent, `tasks.md` scope, existing UI patterns in repo | `ui.md` (flows, screens, states), optional notes in `acceptance.md` | Defines UX expectations that tasks + QA can verify (including accessibility intent) |
| frontend-designer | Design→implementation translator | `ui.md`, codebase components, `tasks.md` | Updates `ui.md` with component breakdown OR writes `architecture.md` UI section | Produces implementable component plan (components/props/states) to reduce dev ambiguity |
| fullstack-developer | Primary builder | `tasks.md`, `acceptance.md`, `ui.md`, `db-migration-plan.md` (if any), repo code | Code changes + tests; updates `architecture.md` when relevant; links in `decisions.md` | Implements tickets; keeps code aligned with `implements:` pointers; adds/updates tests; executes non-DB rollout steps |
| database-administrator (DB Change Guardian) | Migration strategist / safety gate | `spec.md` schema intent (relevant `implements:`), current DB schema, `tasks.md` | `db-migration-plan.md` (expand/contract/backfill/rollback), notes in `decisions.md` | Validates DB changes are production-safe; blocks destructive migration strategies; signs off plan before execution |
| qa | Contract validation + exploratory | `spec.md` (`implements:` pointers), `tasks.md`, `acceptance.md`, running app/test outputs | Updates `acceptance.md` (contract checks + manual scripts), may open issues in `tasks.md` | Validates responses match OpenSpec schemas; defines and re-runs acceptance/regression checks |
| test-automator *(optional but useful)* | Automated test implementer | `acceptance.md`, `tasks.md`, repo test setup | Test files + fixtures; notes in `decisions.md` | Converts acceptance/contract checks into runnable automated tests (unit/integration/e2e) |
| debugger | Root-cause investigator | Failing test logs, QA repro steps, recent diffs | Updates `tasks.md` with “Fix:” tickets; notes in `decisions.md` | Investigates failures; narrows root cause; produces concrete fix tasks (minimal, verifiable) |
| code-reviewer | Quality gate | PR diff, `tasks.md`, `acceptance.md`, relevant `spec.md` nodes | Review notes (commentary); may update `tasks.md` with required fixes; notes in `decisions.md` | Ensures implementation truly satisfies spec + acceptance criteria; flags maintainability/risk |
| security-engineer | Secure-by-design implementer | `tasks.md`, auth model, infra context, threaty endpoints | `security.md` (patterns/decisions), updates `acceptance.md` with security checks | Defines secure implementation patterns + required checks (authz, input validation, secrets) |
| security-auditor | Security reviewer | PR diff, `security.md`, runtime config, dependency list | Findings list (commentary); updates `tasks.md` with remediation tasks; notes in `decisions.md` | Independent security review + required remediation before “done” |
| compliance-engineer (new) | Compliance-by-design | `tasks.md`, `spec.md`, data flows/PHI assumptions | `compliance.md` (requirements), updates `acceptance.md` with compliance checks | Translates healthcare compliance needs into concrete technical requirements + acceptance checks |
| compliance-auditor | Compliance reviewer | `compliance.md`, PR diff, logs/audit trails, data handling | Findings list; updates `tasks.md` with remediation; notes in `decisions.md` | Independent review to ensure compliance requirements are actually implemented |

## File output folder structure

`./ops/ (product + global constraints)`
* product-vision-strategy.md  
* product-principles.md
* tech-stack-constraints.md
* system-invariants.md
* security-baseline.md
* compliance-baseline.md
* db-standards.md

`./ops/build/ (spans multiple versions)`
* architecture.md
* api-standards.md
* testing-strategy.md
* release-policy.md
* decision-log.md (optional)

`./ops/build/v{x}/ (version-level dev artifacts)`
* prd.md (Clavix)
* proposal.md (OpenSpec propose / solution approach)
* design.md (version-level design)
* tasks.md (version-level tasks / epics)
* acceptance.md (version-level gates/criteria)

`./ops/build/v{x}/<feature-name>/ (feature-level artifacts)`
* spec.md (OpenSpec feature spec; source for implements: pointers)
* tasks.md (feature tickets; each ticket includes implements: pointers into spec.md)
* ui.md
* architecture.md
* db-migration-plan.md (if DB changes)
* security.md (if needed beyond baseline)
* compliance.md (if needed beyond baseline)
* acceptance.md (feature-level contract checks + exploratory scripts)
* decisions.md (append-only “what changed/decisions/follow-ups”)
* spec-change-requests.md (when implementation constraints break spec)
