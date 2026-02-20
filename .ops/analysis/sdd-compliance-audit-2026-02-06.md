# SDD & Configuration Compliance Audit Report

**Date:** 2026-02-06
**Scope:** `CLAUDE.md`, `AGENTS.md`, `claude/settings.local.json`, `claude/autonomy-policy.md`, `claude/agents/instructions.md`, `claude/agents/swarm-config.md`, all 19 agent files in `claude/agents/`, `claude/commands/orchestrate.md`
**Exclusions:** Supabase RLS policies, Stripe webhook verification (per user instruction)
**Source of truth:** `CLAUDE.md` (auto-loaded every session)

---

## 1. Executive Summary

**Overall compliance: Good with significant agent-level gaps**

| Priority | Count | Category |
|----------|-------|----------|
| P0 (Security) | 0 | No security vulnerabilities found |
| P1 (SDD Compliance) | 2 | Missing `@` imports + autonomy sections on 16/18 agents; `prd→specs` prerequisite missing from CLAUDE.md |
| P2 (Permissions) | 4 | Deny-list gaps (`git branch -d`, `rm -rf <dir>`, `git checkout .`, `.env` edit variants); autonomy-policy.md narrower than CLAUDE.md on `rm -rf` scope |
| P3 (Consistency) | 3 | `claude/` vs `.claude/` settings.local.json sync gap; swarm template doesn't inject autonomy-policy.md; minor wording differences |

The governance layer (CLAUDE.md, AGENTS.md, instructions.md, swarm-config.md, orchestrate.md, autonomy-policy.md) is **well-aligned** — SDD artifact chain, tier DAG, stop conditions, and parallel execution are consistent across all 6 files. The gap is at the agent level: only 2 of 18 agents (`workflow-orchestrator`, `fullstack-developer`) have `@CLAUDE.md` and `@claude/autonomy-policy.md` imports and an Autonomy section. The remaining 16 agents lack these, meaning they wouldn't inherit the autonomy escalation rules when loaded as standalone agents.

---

## 2. File-by-File Findings

### 2.1 CLAUDE.md (131 lines)

**Correct:**
- Security section is comprehensive and non-negotiable
- Autonomy & Execution has clear "Proceed" vs "Ask" split
- SDD Artifact Flow defines canonical order and 3 prerequisite rules
- Token/Context Rules enforce feature-scoped reads and domain-split loading
- Version Safety prevents cross-version mixing

**Issues:**
- **P1**: Missing `prd.md → specs.md` prerequisite rule. AGENTS.md has it (line 71: "specs.md | prd.md must exist | STOP — ask user") but CLAUDE.md's SDD Artifact Flow section only starts enforcement at `system-design.yaml`. Since CLAUDE.md is auto-loaded every session while AGENTS.md is on-demand, an agent outside the swarm could create `specs.md` without checking for `prd.md`.
- **P2**: Line 38 says `rm -rf` generally requires asking, but `autonomy-policy.md` line 12 narrows this to `-rf /` (root-only). Minor wording inconsistency.

### 2.2 AGENTS.md (311 lines)

**Correct:**
- Agent roster (19 agents) matches actual files in `claude/agents/` — full 1:1 match
- SDD Artifact Flow prerequisite table is complete (5 entries including `prd→specs`)
- Tier Dependency DAG matches swarm-config.md and orchestrate.md exactly
- Stop Conditions correctly scope execution autonomy vs SDD workflow stops
- Parallel Execution Strategy with concrete examples
- Spec Change Request Resolution Workflow is clear
- Definition of Done criteria are testable
- Agent Selection Guide covers all common situations
- References instructions.md and swarm-config.md as supporting refs

**Issues:**
- None found. AGENTS.md is the most complete and well-structured governance file.

### 2.3 claude/settings.local.json

**Correct:**
- 63 allow entries covering all CLAUDE.md "Always Proceed" operations
- 14 deny entries (recently updated from 9) covering most "Always Ask" operations
- Deny overrides allow correctly (e.g., `git stash *` allowed but `git stash drop *` denied)
- MCP servers enabled: supabase, sentry, stripe, context7, playwright, github
- Modern `Bash(command *)` syntax (migrated from deprecated `Bash(command:*)`)

**Issues:**
- **P2**: `git branch -d` (lowercase, safe delete) NOT denied. CLAUDE.md line 34 says "Deleting git branches (`git branch -D` / `-d`)" — both variants should be denied. Only `-D` is currently denied.
- **P2**: `rm -rf <arbitrary-path>` not fully covered. Deny list covers `rm -rf /*`, `rm -rf .*`, `rm -rf .git*` but `rm -rf src/` or `rm -rf node_modules/` would pass through. The broad `Bash(rm *)` allow enables this.
- **P2**: `git checkout .` (discard all unstaged changes) not denied. This is a destructive operation that falls under CLAUDE.md's "Force operations" category.
- **P2**: `.env` modification only partially covered. `Bash(> .env*)` prevents truncation, but `echo "KEY=val" >> .env` or `sed -i '' 's/old/new/' .env` would pass through `Bash(echo *)` and `Bash(sed *)` allows.
- **P3**: `claude/settings.local.json` and `.claude/settings.local.json` are now out of sync. The `claude/` version has modern syntax + expanded deny list while `.claude/` has the old deprecated syntax + 9-entry deny list. If Claude Code reads from `.claude/`, the runtime permissions differ from the version-controlled template.

### 2.4 claude/autonomy-policy.md (67 lines)

**Correct:**
- Decision tree aligns with CLAUDE.md Autonomy & Execution section
- Escalation ladder (Levels 0-4) maps cleanly to CLAUDE.md's "Proceed/Ask" categories
- Parallel Work Policy matches AGENTS.md Parallel Execution Strategy
- Anti-patterns section reinforces autonomous execution (don't ask before routine ops)

**Issues:**
- **P2**: Line 12 specifies destructive force operations as "`--force`, `--hard`, `-rf /`, `chmod 777`" — the `-rf /` is narrower than CLAUDE.md's general `rm -rf`. An agent following autonomy-policy.md might not escalate on `rm -rf important-dir/`.

### 2.5 claude/agents/instructions.md (146 lines)

**Correct:**
- General Guidance section accurately describes the SDD workflow
- Definitions match AGENTS.md strict definitions
- SDD Artifact Flow prerequisite table matches AGENTS.md exactly
- Swarm Orchestration section describes `/orchestrate` flow correctly
- Agent Inputs and Outputs table provides detailed per-agent reads/writes
- Folder structure matches AGENTS.md
- Context-manager exclusivity documented (sole writer to decisions-log.md + implementation-status.md)

**Issues:**
- None found. Fully consistent with AGENTS.md and orchestrate.md.

### 2.6 claude/agents/swarm-config.md (99 lines)

**Correct:**
- Agent Roster table has correct tiers, subagent_type (all `general-purpose`), and auto-detect triggers
- Tier DAG matches AGENTS.md and instructions.md
- Sequential T2 rule documented
- Auto-Detection Keywords are comprehensive and well-categorized
- Teammate Prompt Template is structured
- Message Protocol covers spec-change escalation and context-manager routing
- Correctly excludes workflow-orchestrator (it's the team leader), sdd-specialist, and claude-code-specialist (user-invoked only)
- Always-spawn agents match across all files

**Issues:**
- **P3**: Teammate Prompt Template (lines 66-89) does not include `claude/autonomy-policy.md` content. Spawned agents receive their .md file + workspace context but NOT the autonomy escalation rules. Since CLAUDE.md is auto-loaded for all sessions (including subagents), security and coding rules propagate — but the autonomy-policy.md escalation ladder does not.

### 2.7 claude/commands/orchestrate.md (120 lines)

**Correct:**
- Pre-Flight section enforces all SDD prerequisites with STOP on violation
- Execution steps are detailed and match swarm-config.md tier ordering
- Auto-detection keyword table matches swarm-config.md
- Gate checking includes the database migration pre-T5 check
- Halt Protocol for spec-change-requests
- Completion criteria include Definition of Done verification
- UI system pre-step (check for ui-design-system.md) documented

**Issues:**
- None found. Fully consistent with AGENTS.md and swarm-config.md.

### 2.8 Agent Files — Systematic Findings

#### `@` Imports and Autonomy Sections

| Agent | `@CLAUDE.md` | `@claude/autonomy-policy.md` | Autonomy Section | Category |
|-------|:---:|:---:|:---:|----------|
| workflow-orchestrator | YES | YES | YES | orchestration |
| fullstack-developer | YES | YES | YES | implementation |
| context-manager | NO | NO | NO | orchestration |
| spec-writer | NO | NO | NO | planning |
| architect | NO | NO | NO | planning |
| database-administrator | NO | NO | NO | planning |
| project-task-planner | NO | NO | NO | planning |
| ui-designer | NO | NO | NO | design |
| frontend-designer | NO | NO | NO | design |
| test-automator | NO | NO | NO | quality |
| qa | NO | NO | NO | quality |
| debugger | NO | NO | NO | quality |
| code-reviewer | NO | NO | NO | quality |
| security-engineer | NO | NO | NO | security |
| security-auditor | NO | NO | NO | security |
| compliance-engineer | NO | NO | NO | compliance |
| compliance-auditor | NO | NO | NO | compliance |
| sdd-specialist | NO | NO | NO | meta |
| claude-code-specialist | NO | NO | NO | meta |

**Finding (P1):** 16 of 18 agents lack `@CLAUDE.md` and `@claude/autonomy-policy.md` imports plus an Autonomy section.

**Impact:**
- When spawned as standalone agents (not via swarm), these agents won't have autonomy escalation rules in context
- CLAUDE.md is auto-loaded for all Claude Code sessions (including subagents), so security and coding rules DO propagate at runtime
- However, `autonomy-policy.md` is NOT auto-loaded — agents won't have the decision tree or escalation ladder
- The `claude-code-specialist.md` itself states (line 31): "Agents do NOT auto-inherit CLAUDE.md — must be explicitly referenced via `@` import"

#### Tool Access Audit

All tool assignments are appropriate for each agent's role:

| Agent | Tools | Assessment |
|-------|-------|-----------|
| workflow-orchestrator | Read, Write, Glob, Grep, Bash, **Task** | Correct — only agent with Task tool (spawns agents) |
| fullstack-developer | Read, Write, Edit, Glob, Grep, Bash | Correct — needs Bash for npm/tests |
| test-automator | Read, Write, Edit, Glob, Grep, Bash | Correct — needs Bash for running tests |
| debugger | Read, Edit, Glob, Grep, Bash | Correct — needs Bash for repro steps |
| spec-writer, architect, project-task-planner, database-administrator, context-manager, ui-designer, security-engineer, compliance-engineer, sdd-specialist, claude-code-specialist | Read, Write, Edit, Glob, Grep | Correct — no Bash needed for planning/writing |
| qa, code-reviewer, security-auditor, compliance-auditor, frontend-designer | Read, Edit, Glob, Grep | Correct — validators/reviewers, no Write/Bash (Edit only for checks.yaml/tasks.yaml) |

No agent has tools beyond its role scope. No agent can bypass security through tool access.

#### File Read Scope — Token/Context Rules Compliance

All agents correctly use domain-split files instead of the full product-vision-strategy.md:

| Agent | Vision Doc Reference | Correct Split? |
|-------|---------------------|:-:|
| spec-writer | `.ops/quick-product-vision-strategy.md` + `.ops/security-compliance-baseline.md` | YES |
| architect | `.ops/tech-architecture-baseline.md` | YES |
| database-administrator | `.ops/tech-architecture-baseline.md` | YES |
| security-engineer | `.ops/security-compliance-baseline.md` | YES |
| security-auditor | `.ops/security-compliance-baseline.md` | YES |
| compliance-engineer | `.ops/security-compliance-baseline.md` | YES |
| compliance-auditor | `.ops/security-compliance-baseline.md` | YES |
| sdd-specialist | `.ops/product-vision-strategy.md` (conditional) | ACCEPTABLE (only for governance checks) |
| All others | No vision doc reads | YES (feature-scoped) |

#### SDD Stop Conditions in Agent Files

Every agent that writes artifacts has appropriate stop/escalation conditions:

| Agent | Stop Condition | Present? |
|-------|---------------|:--:|
| spec-writer | Ambiguous info → spec-change-requests | YES |
| architect | Spec/design mismatch → spec-change-requests | YES |
| database-administrator | Destructive migration → STOP for approval | YES |
| project-task-planner | system-design.yaml missing → Hard Stop | YES |
| fullstack-developer | Spec conflict → spec-change-requests, missing db plan → STOP | YES |
| qa | Spec ambiguous → STOP; spec wrong → spec-change-requests | YES |
| test-automator | AC ambiguous → STOP; test/spec conflict → spec-change-requests | YES |
| debugger | Spec mismatch → spec-change-requests; multi-feature → STOP | YES |
| code-reviewer | Spec gap → spec-change-requests; multi-feature PR → STOP | YES |
| security-engineer | Spec conflicts with secure impl → spec-change-requests | YES |
| security-auditor | Findings → remediation tickets (blocking) | YES |
| compliance-engineer | Spec conflicts with compliance → spec-change-requests | YES |
| compliance-auditor | Gaps → remediation tickets (blocking) | YES |

All agents route spec conflicts through `spec-change-requests.yaml` → orchestrator halts. No agent can silently override specs.

---

## 3. Cross-File Consistency Matrix

### SDD Artifact Chain Enforcement

| Rule | CLAUDE.md | AGENTS.md | instructions.md | orchestrate.md |
|------|:-:|:-:|:-:|:-:|
| Canonical order (specs→sysdesign→dbmigration→tasks→buildorder) | YES | YES | YES | YES |
| `prd.md` → `specs.md` prerequisite | **NO** | YES | YES | YES |
| `specs.md` → `system-design.yaml` prerequisite | YES | YES | YES | YES |
| `system-design.yaml` → `db-migration-plan.yaml` prerequisite | YES | YES | YES | YES |
| `specs.md` + `system-design.yaml` → `tasks.yaml` prerequisite | YES | YES | YES | YES |
| All `tasks.yaml` → `build-order.yaml` prerequisite | YES | YES | YES | N/A |

### Autonomy Policy Alignment

| Rule | CLAUDE.md | autonomy-policy.md | settings.local.json |
|------|:-:|:-:|:-:|
| `git push` → ask | YES | YES | DENIED |
| `git branch -D` → ask | YES | YES | DENIED |
| `git branch -d` → ask | YES | Implicit | **NOT DENIED** |
| `.env` modification → ask | YES | YES | Partial (only `>` redirect) |
| `rm -rf` (any path) → ask | YES | **`-rf /` only** | Partial (root + dots only) |
| `git reset --hard` → ask | YES | YES | DENIED |
| `git clean -f/-fd` → ask | YES | YES | DENIED |
| `git checkout -f` → ask | YES | Implicit | DENIED |
| `git checkout .` (discard all) → ask | Implicit | Not covered | **NOT DENIED** |
| `chmod 777` → ask | YES | YES | DENIED |

### Tier DAG Consistency

| Component | AGENTS.md | instructions.md | swarm-config.md | orchestrate.md |
|-----------|:-:|:-:|:-:|:-:|
| T1: context-manager | YES | YES | YES | YES |
| T2: sequential (spec→arch→dba→planner) | YES | YES | YES | YES |
| T3: parallel optionals (UI, security, compliance) | YES | YES | YES | YES |
| T4: frontend-designer (if UI detected) | YES | YES | YES | YES |
| T5: fullstack-dev + test-automator (parallel) | YES | YES | YES | YES |
| T6: qa + reviewers + auditors (parallel) | YES | YES | YES | YES |
| Always-spawn: ctx-mgr, fs-dev, test-auto, qa, code-rev | N/A | YES | YES | YES |

### Context-Manager Exclusivity

| Rule | AGENTS.md | instructions.md | swarm-config.md | context-manager.md |
|------|:-:|:-:|:-:|:-:|
| Sole writer to decisions-log.md | YES | YES | YES | YES |
| Sole writer to implementation-status.md | YES | YES | YES | YES |
| No other agent writes these files | YES | YES | YES | YES |
| Invoked per-tier + on routing keywords | N/A | N/A | YES | YES |

---

## 4. Action Plan

### P1 — SDD Compliance (2 items)

#### P1-1: Add `@` imports and Autonomy sections to 16 agents

**Files:** All agent .md files in `claude/agents/` except `workflow-orchestrator.md` and `fullstack-developer.md`

**What to add to each file** (after the frontmatter and role description line):
```markdown
@CLAUDE.md
@claude/autonomy-policy.md

## Autonomy

Follow `@claude/autonomy-policy.md` escalation ladder. Key rules:
- **Proceed without asking**: file CRUD, `npm install`, tests/builds/lints, branch creation, local commits
- **Stop this task, continue others**: spec conflict → create `spec-change-requests.yaml`, skip to next independent task
- **Stop all work**: security violation, missing critical artifact
```

Customize the Autonomy bullets per agent role where appropriate (e.g., planning agents don't install deps or run builds).

**Why:** The `claude-code-specialist.md` states "Agents do NOT auto-inherit CLAUDE.md — must be explicitly referenced via `@` import." Without these imports, standalone agent invocations won't have the project rules or autonomy escalation ladder in context.

**Affected agents (16):**
context-manager, spec-writer, architect, database-administrator, project-task-planner, ui-designer, frontend-designer, test-automator, qa, debugger, code-reviewer, security-engineer, security-auditor, compliance-engineer, compliance-auditor, sdd-specialist

**Note:** `claude-code-specialist.md` is a special case — it's already built into Claude Code's agent system and may handle imports differently. Add the `@` imports for consistency but verify behavior.

#### P1-2: Add `prd.md → specs.md` prerequisite to CLAUDE.md

**File:** `CLAUDE.md`, SDD Artifact Flow section (line 114)

**Add before line 115:**
```markdown
- Do not create `specs.md` until `prd.md` exists for that build version
```

**Why:** AGENTS.md, instructions.md, and orchestrate.md all enforce this prerequisite, but CLAUDE.md (the always-loaded file) does not. Any agent or direct session that creates specs.md without checking for prd.md would violate SDD without a visible rule stopping it.

### P2 — Permissions (4 items)

#### P2-1: Add `git branch -d` to deny list

**File:** `claude/settings.local.json`

**Add to deny array:**
```json
"Bash(git branch -d *)"
```

**Why:** CLAUDE.md line 34 explicitly says "git branch -D / -d" but only `-D` is denied.

#### P2-2: Add `git checkout .` to deny list

**File:** `claude/settings.local.json`

**Add to deny array:**
```json
"Bash(git checkout . *)",
"Bash(git checkout -- . *)"
```

**Why:** `git checkout .` discards all unstaged changes — a destructive operation falling under CLAUDE.md's force operations category. Currently allowed through `Bash(git checkout *)`.

#### P2-3: Align `rm -rf` scope in autonomy-policy.md with CLAUDE.md

**File:** `claude/autonomy-policy.md`, line 12

**Change:**
```
Is it a destructive force operation (`--force`, `--hard`, `-rf /`, `chmod 777`)?
```
**To:**
```
Is it a destructive force operation (`--force`, `--hard`, `rm -rf`, `chmod 777`)?
```

**Why:** CLAUDE.md says `rm -rf` generally (any target), but autonomy-policy.md narrows it to `-rf /` (root-only). Agents following the autonomy policy might not escalate on `rm -rf important-directory/`.

#### P2-4: Consider adding `.env` edit variants to deny list

**File:** `claude/settings.local.json`

**Optional additions to deny array (evaluate trade-offs):**
```json
"Bash(sed * .env*)",
"Bash(echo * >> .env*)",
"Bash(echo * > .env*)"
```

**Why:** Currently only `Bash(> .env*)` (redirect truncation) is denied. Other `.env` modification methods (`echo >> .env`, `sed -i .env`) pass through. However, this may be intentionally behavioral rather than pattern-enforced — evaluate whether the deny patterns would cause false positives.

### P3 — Consistency (3 items)

#### P3-1: Sync `claude/settings.local.json` → `.claude/settings.local.json`

**Action:** After finalizing `claude/settings.local.json`, copy it to `.claude/settings.local.json` so runtime permissions match the version-controlled template.

**Why:** Currently `claude/settings.local.json` has modern syntax + 14 deny entries while `.claude/settings.local.json` has deprecated syntax + 9 deny entries. If Claude Code reads from `.claude/`, runtime permissions are weaker than intended.

#### P3-2: Include autonomy-policy.md in swarm Teammate Prompt Template

**File:** `claude/agents/swarm-config.md`, Teammate Prompt Template section (lines 66-89)

**Add to the template after the agent system prompt:**
```
Read `claude/autonomy-policy.md` for the escalation ladder before starting work.
```

**Why:** Spawned agents (via Task tool as `general-purpose` subagents) receive only their .md file + workspace context. The `@` imports in agent .md files won't be resolved in spawned subagent prompts. Adding an explicit instruction to read the autonomy policy ensures spawned agents have access to the escalation rules.

#### P3-3: Document the `claude/` vs `.claude/` relationship

**Recommendation:** Add a brief note to `CLAUDE.md` or a README in `claude/` explaining that `claude/settings.local.json` is the version-controlled template and should be synced to `.claude/settings.local.json` for runtime use. This prevents future drift.

---

## 5. Verification Checklist

After implementing the action plan:

- [ ] Every agent .md file has `@CLAUDE.md` and `@claude/autonomy-policy.md` imports
- [ ] Every agent .md file has an Autonomy section
- [ ] CLAUDE.md has `prd.md → specs.md` prerequisite rule
- [ ] `claude/settings.local.json` deny list includes `git branch -d *`
- [ ] `claude/settings.local.json` deny list includes `git checkout . *`
- [ ] `claude/autonomy-policy.md` uses `rm -rf` (not `-rf /`)
- [ ] `claude/settings.local.json` and `.claude/settings.local.json` are in sync
- [ ] swarm-config.md Teammate Prompt Template references autonomy-policy.md
- [ ] JSON valid: `node -e "JSON.parse(require('fs').readFileSync('claude/settings.local.json','utf8'))"`
- [ ] All 19 agent files still parse correctly (YAML frontmatter + markdown body)
