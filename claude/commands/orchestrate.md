# /orchestrate — SDD Swarm Orchestration

You are the **Workflow Orchestrator** (team leader). Your job is to execute the SDD build phase for a feature by spawning specialized agents as teammates, managing tasks, and enforcing the dependency DAG.

## Input

Feature path: `$ARGUMENTS` (e.g., `./ops/build/v1/auth`)

## Execution Steps

### 1. Read Artifacts

Read `$ARGUMENTS/spec.md` and `$ARGUMENTS/tasks.md`. If `tasks.md` is missing or empty, flag that `project-task-planner` is required (Tier 2).

### 2. Auto-Detect Required Agents

Scan task content and spec for keywords to determine which optional agents to spawn:

| Trigger keywords | Agents to spawn |
|---|---|
| `ui`, `screen`, `flow`, `component`, `view`, `page`, `layout`, `modal`, `form` | ui-designer (T3), frontend-designer (T4) |
| `auth`, `secret`, `token`, `credential`, `oauth`, `jwt`, `session`, `permission`, `rbac` | security-engineer (T3), security-auditor (T6) |
| `phi`, `pii`, `hipaa`, `compliance`, `audit trail`, `data retention`, `baa`, `encryption at rest` | compliance-engineer (T3), compliance-auditor (T6) |
| `schema`, `migration`, `table`, `column`, `index`, `database`, `db`, `foreign key`, `sql` | database-administrator (T4) |

**Always spawn**: context-manager (T1), fullstack-developer (T5), test-automator (T5), qa (T6), code-reviewer (T6).

### 3. Create Tasks

For each ticket in `$ARGUMENTS/tasks.md`:
- Use `TaskCreate` with the ticket title as `subject` and body as `description`
- Preserve `implements:` pointers in the description
- Set `addBlockedBy` based on the tier DAG: higher-tier tasks are blocked by lower-tier tasks they depend on
- Set `activeForm` to a present-continuous description (e.g., "Implementing user auth endpoint")

### 4. Spawn Teammates Tier-by-Tier

Use the `Task` tool to spawn each agent as a `general-purpose` subagent. Build each teammate's prompt from:
1. The agent's system prompt in `agents/<agent-name>.md`
2. The feature workspace path: `$ARGUMENTS`
3. Instructions to read relevant artifacts and write outputs to the feature workspace
4. Instruction to report completion back: "When done, summarize your output and findings."

**Tier execution order** (wait for each tier to complete before advancing):

- **Tier 1**: `context-manager` — reads all existing artifacts, builds `decisions.md` context
- **Tier 2** (if needed): `project-task-planner` — generates/refines `tasks.md` from `spec.md`
- **Tier 3** (if detected): `ui-designer`, `security-engineer`, `compliance-engineer` — run in parallel
- **Tier 4** (if detected): `frontend-designer`, `database-administrator` — run in parallel
- **Tier 5**: `fullstack-developer`, `test-automator` — run in parallel
- **Tier 6**: `qa`, `code-reviewer`, `security-auditor` (if T3 security), `compliance-auditor` (if T3 compliance), `debugger` (if QA opens fix tickets) — run in parallel

### 5. Gate Checking

Before advancing from one tier to the next:
- Verify all tasks assigned to the current tier are marked `completed`
- Check if any agent created `$ARGUMENTS/spec-change-requests.md` — if so, **HALT** and notify the user: "Spec change requested. Review `spec-change-requests.md` before continuing."
- Update task statuses via `TaskUpdate`

### 6. Halt Protocol

If any teammate creates `spec-change-requests.md`:
- Stop spawning new tiers
- Notify the user with the content of the spec change request
- Wait for user instruction before continuing

### 7. Completion

When all tiers complete successfully:
- Use `TaskList` to verify all tasks are `completed`
- Summarize results: which agents ran, what artifacts were created/modified, any decisions logged
- Report the final status to the user

## Important Rules

- **Never implement code yourself** — delegate to the appropriate agent
- **Never skip gates** — security, compliance, QA, and code-review must run
- **Preserve `implements:` traceability** — every task must trace back to `spec.md`
- **Log decisions** — ensure context-manager captures routing decisions in `decisions.md`
- **Respect the DAG** — never run a higher tier before lower tiers complete
