# Swarm Configuration — Agent-Teammate Mapping

Reference for `/orchestrate` slash commands. Maps SDD agents to Task tool teammates with tier ordering and auto-detection triggers.

> Note: `workflow-orchestrator` is the team leader executing `/orchestrate` — it reads this file but is not listed as a teammate.

## Agent Roster

| Agent | subagent_type | Tier | Auto-detect trigger | Persistent? |
|---|---|---|---|---|
| context-manager | general-purpose | 1 | Lazy: invoked on routing trigger keywords or at build completion (also on: deviation, scope change, spec break, changed plan, trade-off, constraint, spec-change-request, blocked, architecture decision, migration, security finding, compliance finding) | Yes |
| spec-writer | general-purpose | 2 | `{feature-path}/specs.md` missing OR empty OR `{feature-path}/spec-change-requests.yaml` exists | No |
| architect | general-purpose | 2 | After spec-writer completes OR `system-design.yaml` missing/empty OR user requests system design / architecture | No |
| database-administrator | general-purpose | 2 | Tasks/specs reference database keywords (see below); runs sequentially after architect | No |
| project-task-planner | general-purpose | 2 | `{feature-path}/tasks.yaml` missing OR empty OR `{feature-path}/specs.md` changed since last run | No |
| ui-designer | general-purpose | 3 | Tasks/specs reference UI keywords (see below) | No |
| security-engineer | general-purpose | 3 | Tasks/specs reference security keywords (see below) | No |
| compliance-engineer | general-purpose | 3 | Tasks/specs reference compliance keywords (see below) | No |
| fullstack-developer | general-purpose | 4 | Always | No |
| test-automator | general-purpose | 4 | Always | No |
| qa | general-purpose | 5 | Always | No |
| debugger | general-purpose | 5 | QA opens fix tickets | No |
| code-reviewer | general-purpose | 5 | Always | No |
| security-auditor | general-purpose | 5 | security-engineer was spawned | No |
| compliance-auditor | general-purpose | 5 | compliance-engineer was spawned | No |

**Always spawn**: fullstack-developer (T4), test-automator (T4), qa (T5), code-reviewer (T5).

## Tier Dependency DAG

```
Tier 1: context-manager  (lazy — invoked on triggers or at end)
   |
Tier 2: spec-writer → architect → database-administrator (if DB keywords) → project-task-planner  (sequential, not parallel)
   |
Tier 3: ui-designer, security-engineer, compliance-engineer  (parallel, if detected)
   |
Tier 4: fullstack-developer, test-automator  (parallel)
   |
Tier 5: qa, debugger, code-reviewer, security-auditor, compliance-auditor  (parallel)
```

Tier 2 is **sequential**: spec-writer must complete before architect runs, architect must complete before database-administrator runs (if needed), database-administrator must complete before project-task-planner runs. If `spec-change-requests.yaml` appears after architect, rerun spec-writer for impacted features, then rerun architect before proceeding.

> The tier DAG governs **intra-feature** agent ordering (which agents run in what order for a single feature). For **inter-feature** sequencing (which feature to build first in a multi-feature build), see `.ops/build/v{x}/build-order.yaml` produced by project-task-planner.

## Execution Modes

| Command | Tiers | Use Case |
|---------|-------|----------|
| `/orchestrate:plan` | T1-T2 | Planning only — produce specs, system design, tasks |
| `/orchestrate:build` | T3-T5 | Design + implement + validate (specs/tasks must exist) |
| `/orchestrate:validate` | T5 | QA + review only (implementation must exist) |
| `/orchestrate` | Auto-detect | Routes to appropriate tier range based on artifact state |

### Auto-Detection Logic (for bare `/orchestrate`)

```
IF specs.md missing OR prd.md newer than specs.md:
  mode = full (T1→T5)
ELIF tasks.yaml missing OR specs.md newer than tasks.yaml:
  mode = plan+build (T2→T5)
ELIF implementation files missing OR tasks have status: pending:
  mode = build (T3→T5)
ELSE:
  mode = validate (T5)
```

### T2 Freshness Checks (Smart Skip)

Before spawning each T2 agent, check if output artifact is current:

- spec-writer: SKIP if `specs.md` exists AND is newer than `prd.md`
- architect: SKIP if `system-design.yaml` exists AND is newer than `specs.md`
- database-administrator: SKIP if `db-migration-plan.yaml` exists AND NOT (tasks contain DB keywords AND plan is older than specs)
- project-task-planner: SKIP if `tasks.yaml` exists AND is newer than `specs.md`

## Auto-Detection Keywords

Scan `specs.md` and `tasks.yaml` content for these keywords:

### UI agent (ui-designer)
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
You are the {agent-name}. {content of .claude/agents/<agent-name>.md}

Feature workspace: .ops/build/v{x}/{feature-name}  (aka `{feature-path}`)
Build version: v{x} — all work must stay within this version. Do not mix build versions.

## Communication Rules

- Summaries: ≤ 5 sentences. Bullets, not paragraphs.
- Findings: structured format (tables, YAML, lists). No prose.
- No filler: skip preamble, restatements, "I'll now proceed to...", "Let me analyze..."
- When talking to the user: answer directly. Do not narrate your process.
- When reporting to orchestrator: state what you did, what you produced, and any blockers. Nothing else.

## Pre-loaded Artifacts (DO NOT re-read these files)

### specs.md
{orchestrator inserts actual content of specs.md here — if under 150 lines}

### tasks.yaml
{orchestrator inserts actual content of tasks.yaml here — if under 150 lines}

## Additional Artifacts (read on-demand only if needed for your role)
- .claude/autonomy-policy.md — escalation ladder and decision tree
- .ops/build/v{x}/prd.md — version scope (spec-writer, architect, planner)
- .ops/build/system-design.yaml — architecture reference (architect, db-admin, planner)
- {feature-path}/tasks.yaml — feature tickets (if not pre-loaded above)
- {feature-path}/specs.md — feature requirements (if not pre-loaded above)
- .ops/build/v{x}/db-migration-plan.yaml — DB migration plan (if DB changes)
- .ops/build/v{x}/build-order.yaml — cross-feature build order (if multi-feature)

Perform your role using these artifacts.
Write your outputs to the feature workspace as specified in your role definition.
When complete, summarize what you did and any findings or blockers.
```

**Pre-load rule:** Only pre-load `specs.md` and `tasks.yaml` (read by 5+ agents). If either exceeds 150 lines, include "Read {feature-path}/specs.md" in the on-demand list instead.

## Message Protocol

- Teammates report completion by returning their summary to the orchestrator
- If a teammate identifies a spec violation, it must create `{feature-path}/spec-change-requests.yaml`
- The orchestrator halts on any spec-change-request and notifies the user
- Context-manager is the sole writer to `.ops/build/decisions-log.md` and `.ops/build/v{x}/implementation-status.md`

### Summary Routing (Event-Driven)

Buffer all agent summaries during execution. Route to context-manager only when:
1. **Trigger detected**: Any agent summary contains a routing trigger keyword (deviation, scope change, spec break, spec-change-request, blocked, architecture decision, migration, security finding, compliance finding)
2. **T2 checkpoint**: Always route to context-manager after T2 completes (planning decisions are highest-value to preserve)
3. **Build complete**: After the final tier finishes, route all buffered summaries in a single batch

For clean builds with no triggers, context-manager is invoked once at end (plus T2 checkpoint if T2 ran).
