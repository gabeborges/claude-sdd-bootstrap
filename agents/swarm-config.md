# Swarm Configuration — Agent-Teammate Mapping

Reference for `/orchestrate` slash command. Maps SDD agents to Task tool teammates with tier ordering and auto-detection triggers.

> Note: `workflow-orchestrator` is the team leader executing `/orchestrate` — it reads this file but is not listed as a teammate.

## Agent Roster

| Agent | subagent_type | Tier | Auto-detect trigger | Persistent? |
|---|---|---|---|---|
| context-manager | general-purpose | 1 | Always | Yes |
| knowledge-synthesizer | general-purpose | 1 | Always (also triggered on: deviation, scope change, spec break, changed plan, trade-off, constraint) | No |
| spec-writer | general-purpose | 2 | `{feature-path}/specs.md` missing OR empty OR `{feature-path}/spec-change-requests.yaml` exists | No |
| architect | general-purpose | 2 | After spec-writer completes OR `system-design.yaml` missing/empty OR user requests system design / architecture | No |
| project-task-planner | general-purpose | 2 | `{feature-path}/tasks.yaml` missing OR empty OR `{feature-path}/specs.md` changed since last run | No |
| ui-designer | general-purpose | 3 | Tasks/specs reference UI keywords (see below) | No |
| security-engineer | general-purpose | 3 | Tasks/specs reference security keywords (see below) | No |
| compliance-engineer | general-purpose | 3 | Tasks/specs reference compliance keywords (see below) | No |
| frontend-designer | general-purpose | 4 | ui-designer was spawned | No |
| database-administrator | general-purpose | 4 | Tasks/specs reference database keywords (see below) | No |
| fullstack-developer | general-purpose | 5 | Always | No |
| test-automator | general-purpose | 5 | Always | No |
| qa | general-purpose | 6 | Always | No |
| debugger | general-purpose | 6 | QA opens fix tickets | No |
| code-reviewer | general-purpose | 6 | Always | No |
| security-auditor | general-purpose | 6 | security-engineer was spawned | No |
| compliance-auditor | general-purpose | 6 | compliance-engineer was spawned | No |

**Always spawn**: context-manager (T1), knowledge-synthesizer (T1), fullstack-developer (T5), test-automator (T5), qa (T6), code-reviewer (T6).

## Tier Dependency DAG

```
Tier 1: context-manager, knowledge-synthesizer  (parallel)
   |
Tier 2: spec-writer → architect → project-task-planner  (sequential, not parallel)
   |
Tier 3: ui-designer, security-engineer, compliance-engineer  (parallel, if detected)
   |
Tier 4: frontend-designer, database-administrator  (parallel, if detected)
   |
Tier 5: fullstack-developer, test-automator  (parallel)
   |
Tier 6: qa, debugger, code-reviewer, security-auditor, compliance-auditor  (parallel)
```

Tier 2 is **sequential**: spec-writer must complete before architect runs, architect must complete before project-task-planner runs. If `spec-change-requests.yaml` appears after architect, rerun spec-writer for impacted features, then rerun architect before proceeding.

## Auto-Detection Keywords

Scan `specs.md` and `tasks.yaml` content for these keywords:

### UI agents (ui-designer -> frontend-designer)
`ui`, `screen`, `flow`, `component`, `view`, `page`, `layout`, `modal`, `form`, `button`, `navigation`, `responsive`, `wireframe`

### Security agents (security-engineer -> security-auditor)
`auth`, `secret`, `token`, `credential`, `oauth`, `jwt`, `session`, `permission`, `rbac`, `acl`, `encryption`, `api key`, `password`, `mfa`, `2fa`

### Compliance agents (compliance-engineer -> compliance-auditor)
`phi`, `pii`, `hipaa`, `phipaa`, `pipeda`, `compliance`, `audit trail`, `data retention`, `baa`, `encryption at rest`, `de-identification`, `access log`, `consent`, `gdpr`

### Database agent (database-administrator)
`schema`, `migration`, `table`, `column`, `index`, `database`, `db`, `foreign key`, `sql`, `relation`, `constraint`, `seed`, `backfill`

## Teammate Prompt Template

When spawning a teammate via the `Task` tool, build the prompt as:

```
You are the {agent-name}. {system-prompt-from-.claude/agents/<agent-name>.md}

Feature workspace: .ops/build/v{x}/{feature-name}  (aka `{feature-path}`)
Build version: v{x} — all work must stay within this version. Do not mix build versions.

## Context

Read the following artifacts before starting:
1. .ops/build/v{x}/prd.md — version scope and acceptance intent
2. {feature-path}/specs.md — feature requirements
3. .ops/build/system-design.yaml — architecture reference
4. {feature-path}/tasks.yaml — feature tickets (if present)

Perform your role using these artifacts.
Write your outputs to the feature workspace as specified in your role definition.
When complete, summarize what you did and any findings or blockers.
```

## Message Protocol

- Teammates report completion by returning their summary to the orchestrator
- If a teammate identifies a spec violation, it must create `{feature-path}/spec-change-requests.yaml`
- The orchestrator halts on any spec-change-request and notifies the user
- Context-manager appends all decisions and state changes to `.ops/build/decisions-log.md`
- If any agent reports deviation, scope change, spec break, changed plan, trade-off, or constraint, the orchestrator runs knowledge-synthesizer to update build logs
