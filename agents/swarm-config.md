# Swarm Configuration — Agent↔Teammate Mapping

Reference for `/orchestrate` slash command. Maps SDD agents to Task tool teammates with tier ordering and auto-detection triggers.

## Agent Roster

| Agent | subagent_type | Tier | Auto-detect trigger | Persistent? |
|---|---|---|---|---|
| context-manager | general-purpose | 1 | Always | Yes |
| project-task-planner | general-purpose | 2 | `tasks.md` empty or missing | No |
| ui-designer | general-purpose | 3 | Tasks reference UI/screens/flows/components | No |
| security-engineer | general-purpose | 3 | Tasks reference auth/secrets/tokens/credentials | No |
| compliance-engineer | general-purpose | 3 | Tasks reference PHI/PII/HIPAA/data-retention | No |
| frontend-designer | general-purpose | 4 | ui-designer was spawned | No |
| database-administrator | general-purpose | 4 | Tasks reference schema/migration/table/database | No |
| fullstack-developer | general-purpose | 5 | Always | No |
| test-automator | general-purpose | 5 | Always | No |
| qa | general-purpose | 6 | Always | No |
| debugger | general-purpose | 6 | QA opens fix tickets | No |
| code-reviewer | general-purpose | 6 | Always | No |
| security-auditor | general-purpose | 6 | security-engineer was spawned | No |
| compliance-auditor | general-purpose | 6 | compliance-engineer was spawned | No |

## Tier Dependency DAG

```
T1: context-manager
 └→ T2: project-task-planner (conditional)
     └→ T3: ui-designer | security-engineer | compliance-engineer (parallel, conditional)
         └→ T4: frontend-designer | database-administrator (parallel, conditional)
             └→ T5: fullstack-developer | test-automator (parallel)
                 └→ T6: qa | code-reviewer | security-auditor | compliance-auditor | debugger (parallel)
```

## Auto-Detection Keywords

### UI agents (ui-designer → frontend-designer)
`ui`, `screen`, `flow`, `component`, `view`, `page`, `layout`, `modal`, `form`, `button`, `navigation`, `responsive`, `wireframe`

### Security agents (security-engineer → security-auditor)
`auth`, `secret`, `token`, `credential`, `oauth`, `jwt`, `session`, `permission`, `rbac`, `acl`, `encryption`, `api key`, `password`, `mfa`, `2fa`

### Compliance agents (compliance-engineer → compliance-auditor)
`phi`, `pii`, `hipaa`, `compliance`, `audit trail`, `data retention`, `baa`, `encryption at rest`, `de-identification`, `access log`, `consent`, `gdpr`

### Database agent (database-administrator)
`schema`, `migration`, `table`, `column`, `index`, `database`, `db`, `foreign key`, `sql`, `relation`, `constraint`, `seed`, `backfill`

## Teammate Prompt Template

When spawning a teammate via the `Task` tool, build the prompt as:

```
You are the {agent-name}. {system-prompt-from-agents/<agent-name>.md}

Feature workspace: {feature-path}

Read the relevant artifacts in the feature workspace and perform your role.
Write your outputs to the feature workspace as specified in your role definition.
When complete, summarize what you did and any findings or blockers.
```

## Message Protocol

- Teammates report completion by returning their summary to the orchestrator
- If a teammate identifies a spec violation, it must create `{feature-path}/spec-change-requests.md`
- The orchestrator halts on any spec-change-request and notifies the user
- Context-manager appends all decisions and state changes to `{feature-path}/decisions.md`
