# Autonomy Policy — Decision Framework for Claude Code Agents

Reference document for agents. Import via `@claude/autonomy-policy.md`.

## Should I Ask or Act? — Decision Tree

Answer in order. First YES wins.

| Question | Answer | Action |
|----------|--------|--------|
| Does it modify external/shared systems (production, remote repo, webhooks, third-party APIs)? | YES | **Ask** |
| Is it a destructive force operation (`--force`, `--hard`, `-rf /`, `chmod 777`)? | YES | **Ask** |
| Does it conflict with specs or security rules (CLAUDE.md § Security)? | YES | **Stop** and escalate |
| Does it modify `.env` files or environment variables? | YES | **Ask** |
| Is it within project directory AND routine dev operation (file CRUD, deps, tests, builds, local commits, branches)? | YES | **Act** |
| Am I unsure about correctness (ambiguous spec, unclear requirements)? | YES | **Act** with smallest reasonable scope, flag ambiguity |
| Am I unsure about safety (could break things, unclear scope)? | YES | **Ask** |
| Default | — | **Act** within stated scope |

## Parallel Work Policy

- **Before starting work**: Identify all independent work items (tasks without `blockedBy` dependencies, agents in the same tier, features without cross-dependencies)
- **Use Task tool**: Spawn parallel agents/tasks for independent items in a single message — do NOT spawn sequentially if they can run in parallel
- **When blocked**: Flag blocker immediately with brief explanation, continue ALL other independent work — never wait idle
- **Reporting**: When returning status, clearly separate completed vs. in-progress vs. blocked items

## Escalation Ladder (Ordered by Severity)

### Level 0: Act Autonomously
- File create/edit/delete within project
- Install dependencies (`npm install`)
- Run tests, builds, lints
- Create/switch git branches
- Stage and commit locally
- Run dev database migrations
- Routine read-only commands

### Level 1: Act and Inform
- Non-obvious technical decisions within scope (chose X over Y because...)
- Inferred missing details from context
- Applied smallest reasonable scope when requirements ambiguous

### Level 2: Flag and Continue
- One task blocked on user input — flag it, continue others
- Missing optional documentation
- Non-critical warnings in tests/builds

### Level 3: Stop This Task, Continue Others
- Spec conflict — cannot implement as written (create `spec-change-requests.yaml`, skip this task, work on independent tasks)
- Missing prerequisite for this specific task (need DB migration plan, UI design, etc.)
- Ambiguity affecting correctness for this task

### Level 4: Stop All Work
- Security risk detected (credentials exposure, auth bypass, RLS violation)
- Missing critical artifact blocking entire workflow (no `specs.md`, no `system-design.yaml` when required)
- Destructive action affecting shared state (delete production data, force push, etc.)

## Anti-Patterns (What NOT to Do)

- ❌ Don't ask "should I create this file?" — just create it
- ❌ Don't ask "should I install this dependency?" — just install it (if in task/spec)
- ❌ Don't ask "should I run tests?" — just run them
- ❌ Don't ask "should I commit locally?" — just commit with clear message
- ❌ Don't halt all work because one task needs input — flag it and continue others
- ❌ Don't wait for permission on routine operations — assume permission within scope
- ❌ Don't implement tasks sequentially if they can run in parallel
