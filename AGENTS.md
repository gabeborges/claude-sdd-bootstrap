# AGENTS.md — Agent Coordination & SDD Workflow

On-demand reference for orchestrator agents. Security and coding rules live in `CLAUDE.md` (auto-loaded every session).

---

## SDD Artifacts (Priority Order)

Agents MUST consult these artifacts before coding:

| Priority | Artifact | Path | Purpose |
|----------|----------|------|---------|
| 1 | Product Vision (canonical) | `.ops/product-vision-strategy.md` | Full vision — cross-domain analysis only. Regenerate splits with `/vision:distill` |
| 1a | Product Vision (distilled) | `.ops/quick-product-vision-strategy.md` | Product context: vision, pillars, non-goals, AI strategy (§1–§7, §12) |
| 1b | Security/Compliance Baseline | `.ops/security-compliance-baseline.md` | Security posture, compliance, AI boundaries, tech non-goals (§10 partial, §11, §12, §15, §16) |
| 1c | Architecture Baseline | `.ops/tech-architecture-baseline.md` | Architecture, data, integration, scalability, tech non-goals (§8–§10, §12–§16) |
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
        └── <feature-name>/
            ├── specs.md
            ├── tasks.yaml
            ├── db-migration-plan.yaml (conditional — required if DB changes detected)
            └── (optional) checks.yaml, spec-change-requests.yaml
```

## Workflow Discipline

### SDD Artifact Flow
Canonical order: `specs.md` (spec-writer) → `system-design.yaml` (architect) → `db-migration-plan.yaml` (database-administrator, if DB changes) → `tasks.yaml` (project-task-planner)

**Prerequisite rules (stop conditions):**

| Artifact | Prerequisite | On Violation |
|---|---|---|
| `specs.md` | `prd.md` must exist for that version | STOP — ask user |
| `system-design.yaml` | `specs.md` must exist for that feature | STOP — run spec-writer first |
| `db-migration-plan.yaml` | `system-design.yaml` complete AND specs contain DB keywords | STOP — run architect first |
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
| Context Manager | Decision log + deviation logger (sole writer) | orchestration | `.claude/agents/context-manager.md` |
| Spec Writer | Spec authoring + feature breakdown | planning | `.claude/agents/spec-writer.md` |
| Architect | System design maintainer | planning | `.claude/agents/architect.md` |
| Database Administrator | Migration strategist / safety gate | planning | `.claude/agents/database-administrator.md` |
| Project Task Planner | Spec handoff / ticket writer | planning | `.claude/agents/project-task-planner.md` |
| UI Designer | UX intent + flows | design | `.claude/agents/ui-designer.md` |
| Frontend Designer | Design-to-implementation translator | design | `.claude/agents/frontend-designer.md` |
| Fullstack Developer | Primary builder | implementation | `.claude/agents/fullstack-developer.md` |
| QA | Contract validation + exploratory | quality | `.claude/agents/qa.md` |
| Test Automator | Automated test implementer | quality | `.claude/agents/test-automator.md` |
| Debugger | Root-cause investigator | quality | `.claude/agents/debugger.md` |
| Code Reviewer | Quality gate | quality | `.claude/agents/code-reviewer.md` |
| Security Engineer | Secure-by-design implementer | security | `.claude/agents/security-engineer.md` |
| Security Auditor | Independent security reviewer | security | `.claude/agents/security-auditor.md` |
| Compliance Engineer | Compliance-by-design | compliance | `.claude/agents/compliance-engineer.md` |
| Compliance Auditor | Compliance reviewer | compliance | `.claude/agents/compliance-auditor.md` |
| SDD Specialist | SDD workflow config creator/auditor | meta | `.claude/agents/sdd-specialist.md` |
| Claude Code Specialist | CLAUDE.md/AGENTS.md/agent/skill optimizer | meta | `.claude/agents/claude-code-specialist.md` |

## By Category

### Orchestration
- **Workflow Orchestrator** — Enforces SDD sequence, triggers agents, routes spec breaks
- **Context Manager** — Sole writer to decisions-log.md and implementation-status.md; records decisions, state changes, deviations

### Planning
- **Spec Writer** — Spec authoring + feature breakdown
- **Architect** — Maintains `system-design.yaml` from specs; identifies spec/architecture mismatches
- **Database Administrator** — DB change guardian, expand/contract migration plans (runs after architect, before task planner)
- **Project Task Planner** — Parses specs into feature `tasks.yaml` with `implements:` pointers

### Design
- **UI Designer** — UX flows, screens, states, accessibility
- **Frontend Designer** — Component breakdown (props/states) from UI specs

### Implementation
- **Fullstack Developer** — Primary code writer, implements feature `tasks.yaml` with tests

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

### Meta (User-Invoked Only)
- **SDD Specialist** — Creates and audits SDD workflow configuration files (agent definitions, instructions.md, AGENTS.md, swarm-config.md, skills). Not part of automated swarm.
- **Claude Code Specialist** — Creates and optimizes CLAUDE.md, AGENTS.md, agent files, and skills. Not part of automated swarm.

## Dependency DAG

```
Tier 1: workflow-orchestrator, context-manager
   ↓
Tier 2: spec-writer → architect → database-administrator (if DB keywords) → project-task-planner  (sequential)
   ↓
Tier 3: ui-designer, security-engineer, compliance-engineer  (parallel, if detected)
   ↓
Tier 4: frontend-designer  (if detected)
   ↓
Tier 5: fullstack-developer, test-automator  (parallel)
   ↓
Tier 6: qa, debugger, code-reviewer, security-auditor, compliance-auditor  (parallel)
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

## Spec Change Request Resolution Workflow

When any agent creates `spec-change-requests.yaml`:

1. **Halt**: Workflow-orchestrator stops tier progression immediately
2. **Review**: Human or spec-writer reviews the requests in `spec-change-requests.yaml`
3. **Update specs**: `specs.md` is updated to resolve the conflicts
4. **Regenerate downstream artifacts**: Affected downstream artifacts are regenerated in order:
   - `system-design.yaml` (architect reruns if spec changes affect architecture)
   - `db-migration-plan.yaml` (database-administrator reruns if spec changes affect schema)
   - `tasks.yaml` (project-task-planner reruns to reflect updated specs/design)
5. **Resume**: Workflow resumes from the first affected tier

## Swarm Orchestration

Use `/orchestrate <feature-path>` to run the full SDD build phase automatically.

Feature workspace: `.ops/build/v{x}/<feature-name>/`

The workflow-orchestrator spawns agents tier-by-tier and auto-detects which optional agents are needed from `tasks.yaml` / `specs.md` content.
See [.claude/agents/swarm-config.md](.claude/agents/swarm-config.md) for agent-teammate mapping and keyword triggers.

## Supporting References

- Agent roles, inputs, outputs: [.claude/agents/instructions.md](.claude/agents/instructions.md)
- Swarm orchestration config: [.claude/agents/swarm-config.md](.claude/agents/swarm-config.md)
