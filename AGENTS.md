<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# SDD Subagent Index

This file indexes all subagents in the SDD (Spec-Driven Development) workflow. Each agent has a dedicated definition file in `agents/`.

## Quick Reference

| Agent | Role | Category | File |
|-------|------|----------|------|
| Workflow Orchestrator | Orchestration + routing | orchestration | [agents/workflow-orchestrator.md](agents/workflow-orchestrator.md) |
| Context Manager | Memory + decision log | orchestration | [agents/context-manager.md](agents/context-manager.md) |
| Project Task Planner | Spec handoff / ticket writer | planning | [agents/project-task-planner.md](agents/project-task-planner.md) |
| UI Designer | UX intent + flows | design | [agents/ui-designer.md](agents/ui-designer.md) |
| Frontend Designer | Design→implementation translator | design | [agents/frontend-designer.md](agents/frontend-designer.md) |
| Fullstack Developer | Primary builder | implementation | [agents/fullstack-developer.md](agents/fullstack-developer.md) |
| Database Administrator | Migration strategist / safety gate | implementation | [agents/database-administrator.md](agents/database-administrator.md) |
| QA | Contract validation + exploratory | quality | [agents/qa.md](agents/qa.md) |
| Test Automator | Automated test implementer | quality | [agents/test-automator.md](agents/test-automator.md) |
| Debugger | Root-cause investigator | quality | [agents/debugger.md](agents/debugger.md) |
| Code Reviewer | Quality gate | quality | [agents/code-reviewer.md](agents/code-reviewer.md) |
| Security Engineer | Secure-by-design implementer | security | [agents/security-engineer.md](agents/security-engineer.md) |
| Security Auditor | Independent security reviewer | security | [agents/security-auditor.md](agents/security-auditor.md) |
| Compliance Engineer | Compliance-by-design | compliance | [agents/compliance-engineer.md](agents/compliance-engineer.md) |
| Compliance Auditor | Compliance reviewer | compliance | [agents/compliance-auditor.md](agents/compliance-auditor.md) |

## By Category

### Orchestration
- **Workflow Orchestrator** — Enforces SDD sequence, triggers agents, routes spec breaks
- **Context Manager** — Append-only decision log, resumable state

### Planning
- **Project Task Planner** — Parses OpenSpec into `tasks.md` with `implements:` pointers

### Design
- **UI Designer** — UX flows, screens, states, accessibility
- **Frontend Designer** — Component breakdown (props/states) from `ui.md`

### Implementation
- **Fullstack Developer** — Primary code writer, implements tasks with tests
- **Database Administrator** — DB Change Guardian, expand/contract migration plans

### Quality
- **QA** — Contract validation against OpenSpec schemas
- **Test Automator** — Converts acceptance checks into automated tests
- **Debugger** — Root-cause analysis, produces fix tickets
- **Code Reviewer** — PR quality gate, spec compliance verification

### Security
- **Security Engineer** — Defines secure-by-design patterns in `security.md`
- **Security Auditor** — Independent security review, OWASP checks

### Compliance
- **Compliance Engineer** — Translates compliance needs into technical requirements
- **Compliance Auditor** — Independent compliance verification

## Dependency DAG

```
Tier 1: workflow-orchestrator, context-manager
   ↓
Tier 2: project-task-planner
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

| Situation | Agent to invoke |
|-----------|-----------------|
| Starting a new feature | workflow-orchestrator |
| Spec is ready, need tasks | project-task-planner |
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

## Swarm Orchestration

Use `/orchestrate <feature-path>` to run the full SDD build phase automatically. The workflow-orchestrator spawns agents as teammates tier-by-tier, auto-detecting which optional agents are needed from task/spec content. See [agents/swarm-config.md](agents/swarm-config.md) for the full agent↔teammate mapping and keyword triggers.

## Source of Truth

- Agent roles, inputs, outputs: [agents/instructions.md](agents/instructions.md)
- Agent template format: [agentsmd/agents.md](https://github.com/agentsmd/agents.md)
- Swarm orchestration config: [agents/swarm-config.md](agents/swarm-config.md)
