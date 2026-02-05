# AGENTS.md — Agent Coordination & SDD Workflow

On-demand reference for orchestrator agents. Security and coding rules live in `CLAUDE.md` (auto-loaded every session).

---

## SDD Artifacts (Priority Order)

Agents MUST consult these artifacts before coding:

| Priority | Artifact | Path | Purpose |
|----------|----------|------|---------|
| 1 | Product Vision | `.ops/product-vision-strategy.md` | High-level product context (Clavix) |
| 2 | UI Design System | `.ops/ui-design-system.md` | UI source of truth (if present) |
| 3 | System Design | `.ops/build/system-design.yaml` | Architecture reference |
| 4 | PRD | `.ops/build/v{x}/prd.md` | Version scope + acceptance intent |
| 5 | Specs | `.ops/build/v{x}/<feature-name>/specs.md` | Requirements + acceptance criteria (source of truth) |
| 6 | Tasks | `.ops/build/v{x}/<feature-name>/tasks.yaml` | Feature tickets with `implements:` pointers into specs |

**Do not proceed with coding if any required artifact is missing or ambiguous. Ask questions.**

### Strict Definitions
- `system-design.yaml` = architecture reference spanning multiple product versions in `.ops/build/system-design.yaml`
- `prd.md` = build level plan in `.ops/build/v{x}/prd.md`
- `implementation-status.md` = build level progress tracker in `.ops/build/v{x}/implementation-status.md`
- `specs.md` = feature requirements + acceptance criteria in `.ops/build/v{x}/<feature-name>/specs.md`
- `tasks.yaml` = feature tickets in `.ops/build/v{x}/<feature-name>/tasks.yaml`

### Folder Structure
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

## Workflow Discipline

### SDD Artifact Flow
Canonical order: `specs.md` (spec-writer) → `system-design.yaml` (architect) → `tasks.yaml` (project-task-planner)

**Prerequisite rules (stop conditions):**

| Artifact | Prerequisite | On Violation |
|---|---|---|
| `specs.md` | `prd.md` must exist for that version | STOP — ask user |
| `system-design.yaml` | `specs.md` must exist for that feature | STOP — run spec-writer first |
| `tasks.yaml` | Both `specs.md` AND `system-design.yaml` must exist | STOP — run architect first |

### Spec vs Implementation
- Write `specs.md` and `implementation-status.md` only in the proposal phase
- Only implement code from tasks in `tasks.yaml` (feature-level)

### Spec-Change Requests
If implementation constraints conflict with the spec:
- Create `.ops/build/v{x}/<feature-name>/spec-change-requests.yaml` (brief, factual)
- Stop and escalate to the orchestrator/user

## Stop Conditions (Ask Before Proceeding)

STOP and ask (do not guess) if:
- Target build version `v{x}` is unclear
- `prd.md` is missing and specs are about to be created
- `specs.md` is missing, contradictory, or lacks acceptance scenarios
- `system-design.yaml` is missing and tasks are about to be created (architect must run first)
- `tasks.yaml` is missing or any ticket lacks a valid `implements:` pointer
- Implementation would require changing requirements or acceptance criteria
- A security/compliance risk exists (see security rules in CLAUDE.md)
- Stripe / OAuth / Supabase settings are required but not specified

## Definition of Done

A ticket is "done" ONLY when:
- Tests pass locally (Vitest + Playwright as applicable)
- Tests added/updated for new behavior (proportional to risk)
- All relevant `specs.md` scenarios are satisfied
- Code reviewed (peer/agent reviewer per workflow)

When finishing work, report:
- What changed (files/modules)
- Which `specs.md` scenarios were satisfied (via `implements:` pointers)
- What validation was performed (commands run + outcomes)

---

## Agent Roster

| Agent | Role | Category | File |
|-------|------|----------|------|
| Workflow Orchestrator | Orchestration + routing | orchestration | `.claude/agents/workflow-orchestrator.md` |
| Context Manager | Memory + decision log | orchestration | `.claude/agents/context-manager.md` |
| Knowledge Synthesizer | Deviation logger + build knowledge compactor | orchestration | `.claude/agents/knowledge-synthesizer.md` |
| Spec Writer | Spec authoring + feature breakdown | planning | `.claude/agents/spec-writer.md` |
| Architect | System design maintainer | planning | `.claude/agents/architect.md` |
| Project Task Planner | Spec handoff / ticket writer | planning | `.claude/agents/project-task-planner.md` |
| UI Designer | UX intent + flows | design | `.claude/agents/ui-designer.md` |
| Frontend Designer | Design-to-implementation translator | design | `.claude/agents/frontend-designer.md` |
| Fullstack Developer | Primary builder | implementation | `.claude/agents/fullstack-developer.md` |
| Database Administrator | Migration strategist / safety gate | implementation | `.claude/agents/database-administrator.md` |
| QA | Contract validation + exploratory | quality | `.claude/agents/qa.md` |
| Test Automator | Automated test implementer | quality | `.claude/agents/test-automator.md` |
| Debugger | Root-cause investigator | quality | `.claude/agents/debugger.md` |
| Code Reviewer | Quality gate | quality | `.claude/agents/code-reviewer.md` |
| Security Engineer | Secure-by-design implementer | security | `.claude/agents/security-engineer.md` |
| Security Auditor | Independent security reviewer | security | `.claude/agents/security-auditor.md` |
| Compliance Engineer | Compliance-by-design | compliance | `.claude/agents/compliance-engineer.md` |
| Compliance Auditor | Compliance reviewer | compliance | `.claude/agents/compliance-auditor.md` |

## By Category

### Orchestration
- **Workflow Orchestrator** — Enforces SDD sequence, triggers agents, routes spec breaks
- **Context Manager** — Append-only decision log, resumable state
- **Knowledge Synthesizer** — Deviation logger + build knowledge compactor

### Planning
- **Spec Writer** — Spec authoring + feature breakdown
- **Architect** — Maintains `system-design.yaml` from specs; identifies spec/architecture mismatches
- **Project Task Planner** — Parses specs into feature `tasks.yaml` with `implements:` pointers

### Design
- **UI Designer** — UX flows, screens, states, accessibility
- **Frontend Designer** — Component breakdown (props/states) from UI specs

### Implementation
- **Fullstack Developer** — Primary code writer, implements feature `tasks.yaml` with tests
- **Database Administrator** — DB change guardian, expand/contract migration plans

### Quality
- **QA** — Contract validation against `specs.md` scenarios
- **Test Automator** — Converts `specs.md` scenarios into automated tests
- **Debugger** — Root-cause analysis, produces fix tickets
- **Code Reviewer** — PR quality gate, spec compliance verification

### Security & Compliance
- **Security Engineer** — Defines secure-by-design patterns
- **Security Auditor** — Independent security review, OWASP checks
- **Compliance Engineer** — Translates compliance needs into technical requirements
- **Compliance Auditor** — Independent compliance verification

## Dependency DAG

```
Tier 1: workflow-orchestrator, context-manager
   ↓
Tier 2: spec-writer → architect → project-task-planner
   ↓
Tier 3: ui-designer, security-engineer, compliance-engineer
   ↓
Tier 4: frontend-designer, database-administrator
   ↓
Tier 5: fullstack-developer, test-automator
   ↓
Tier 6: qa, debugger, code-reviewer, security-auditor, compliance-auditor
```

## Agent Selection Guide

| Situation | Agent to Invoke |
|-----------|-----------------|
| Starting a new feature | workflow-orchestrator |
| Spec missing/needs writing | spec-writer |
| System design needed / architecture decisions | architect |
| Spec ready, need system design before tasks | architect |
| Spec ready, need feature tasks | project-task-planner |
| Feature has UI | ui-designer → frontend-designer |
| Feature has DB changes | database-administrator |
| Ready to implement | fullstack-developer |
| Implementation done, need validation | qa + test-automator |
| Tests failing | debugger |
| PR ready for review | code-reviewer |
| Security-sensitive feature | security-engineer → security-auditor |
| Compliance-sensitive feature | compliance-engineer → compliance-auditor |
| Need to record a decision | context-manager |
| Spec can't be implemented as-is | workflow-orchestrator (creates spec-change-request) |

**UI system rule:** Before any UI work, check for `.ops/ui-design-system.md`. If missing, run `/interface-design:init`. If present, `ui-designer` and `frontend-designer` MUST follow it. Use `frontend-design` skill for simple one-off pages/components.

## Swarm Orchestration

Use `/orchestrate <feature-path>` to run the full SDD build phase automatically.

Feature workspace: `.ops/build/v{x}/<feature-name>/`

The workflow-orchestrator spawns agents tier-by-tier and auto-detects which optional agents are needed from `tasks.yaml` / `specs.md` content.
See [.claude/agents/swarm-config.md](.claude/agents/swarm-config.md) for agent-teammate mapping and keyword triggers.

## Supporting References

- Agent roles, inputs, outputs: [.claude/agents/instructions.md](.claude/agents/instructions.md)
- Swarm orchestration config: [.claude/agents/swarm-config.md](.claude/agents/swarm-config.md)
