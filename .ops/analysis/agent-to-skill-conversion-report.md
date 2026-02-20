# Agent-to-Skill Conversion Analysis

**Generated**: 2026-02-08
**Scope**: Evaluate all agents in `claude/agents/` for potential removal or conversion to skills
**Agent performing analysis**: SDD Specialist

---

## Executive Summary

**19 agent files** analyzed. **17 are part of the swarm tier DAG** and must remain agents (the orchestrator spawns them via Task tool with isolated context windows). **2 are user-invoked only** and are strong candidates for conversion to skill commands.

Additionally, those 2 user-invoked agents (`sdd-specialist` and `claude-code-specialist`) have **significant responsibility overlap** — both create agent files, both create skills, both update AGENTS.md. Recommendation: merge and convert into a unified skill group.

---

## Agent Catalog

| # | Agent File | Category | Tier | Tools | Swarm Role | User-Invoked? |
|---|-----------|----------|------|-------|-----------|---------------|
| 1 | `workflow-orchestrator.md` | orchestration | T1 | Read, Write, Glob, Grep, Bash, Task | Team leader, enforces tier DAG | No (entry via `/orchestrate`) |
| 2 | `context-manager.md` | orchestration | T1 | Read, Write, Edit, Glob, Grep | Persistent, sole writer to decisions-log.md | No |
| 3 | `spec-writer.md` | planning | T2 | Read, Write, Edit, Glob, Grep | Creates specs.md from PRD | No |
| 4 | `architect.md` | planning | T2 | Read, Write, Edit, Glob, Grep | Maintains system-design.yaml | No |
| 5 | `database-administrator.md` | planning | T2 | Read, Write, Edit, Glob, Grep | Plans DB migrations (conditional) | No |
| 6 | `project-task-planner.md` | planning | T2 | Read, Write, Edit, Glob, Grep | Generates tasks.yaml | No |
| 7 | `ui-designer.md` | design | T3 | Read, Write, Edit, Glob, Grep | Creates UX flows in ui.md (conditional) | No |
| 8 | `security-engineer.md` | security | T3 | Read, Write, Edit, Glob, Grep | Defines security.yaml patterns (conditional) | No |
| 9 | `compliance-engineer.md` | compliance | T3 | Read, Write, Edit, Glob, Grep | Creates compliance.yaml (conditional) | No |
| 10 | `frontend-designer.md` | design | T4 | Read, Edit, Glob, Grep | Component architecture in ui.md (conditional) | No |
| 11 | `fullstack-developer.md` | implementation | T5 | Read, Write, Edit, Glob, Grep, Bash | Implements tasks with code + tests | No |
| 12 | `test-automator.md` | quality | T5 | Read, Write, Edit, Glob, Grep, Bash | Converts spec ACs to automated tests | No |
| 13 | `qa.md` | quality | T6 | Read, Edit, Glob, Grep | Validates implementation against spec contracts | No |
| 14 | `debugger.md` | quality | T6 | Read, Edit, Glob, Grep, Bash | Root-cause analysis, creates fix tickets (conditional) | No |
| 15 | `code-reviewer.md` | quality | T6 | Read, Edit, Glob, Grep | PR quality gate | No |
| 16 | `security-auditor.md` | security | T6 | Read, Edit, Glob, Grep | Independent security review (conditional) | No |
| 17 | `compliance-auditor.md` | compliance | T6 | Read, Edit, Glob, Grep | Independent compliance review (conditional) | No |
| 18 | `sdd-specialist.md` | meta | — | Read, Write, Edit, Glob, Grep | **NOT in swarm** | **Yes (user-invoked only)** |
| 19 | `claude-code-specialist.md` | meta | — | Read, Write, Edit, Glob, Grep | **NOT in swarm** | **Yes (user-invoked only)** |

---

## Recommendation Table

| Agent File | Purpose | Verdict | Rationale |
|-----------|---------|---------|-----------|
| `workflow-orchestrator.md` | Enforces SDD sequence, spawns agents, validates gates | **KEEP** | Core orchestrator; uses Task tool for multi-agent spawning; irreplaceable |
| `context-manager.md` | Records decisions, state changes, deviations | **KEEP** | Persistent across tiers; sole writer to decisions-log.md + implementation-status.md; stateful |
| `spec-writer.md` | Creates specs.md from PRD | **KEEP** | Core T2 planning; multi-step with escalation; referenced by architect + project-task-planner |
| `architect.md` | Maintains system-design.yaml | **KEEP** | Core T2 planning; maintains evolving cross-version artifact; referenced by downstream agents |
| `database-administrator.md` | Plans safe DB migrations | **KEEP** | T2 swarm agent; specialized domain expertise; coexists with `/db:plan` skill (user-invoked equivalent) |
| `project-task-planner.md` | Generates tasks.yaml from specs | **KEEP** | Core T2 planning; cross-feature build-order.yaml generation; referenced by orchestrator |
| `ui-designer.md` | Creates UX flows, screen definitions, states | **KEEP** | T3 swarm agent; frontend-designer (T4) depends on its output; chain dependency |
| `security-engineer.md` | Defines security patterns, creates security.yaml | **KEEP** | T3 swarm agent; security-auditor (T6) depends on its activation; chain dependency |
| `compliance-engineer.md` | Creates compliance.yaml, adds compliance checks | **KEEP** | T3 swarm agent; compliance-auditor (T6) depends on its activation; chain dependency |
| `frontend-designer.md` | Translates UX flows into component architecture | **KEEP** | T4 swarm agent; bridges ui-designer output → fullstack-developer input |
| `fullstack-developer.md` | Implements tasks with production code + tests | **KEEP** | Core T5 builder; always-spawn; uses Bash for builds/tests; multi-step |
| `test-automator.md` | Converts spec ACs into automated tests | **KEEP** | Core T5 quality; always-spawn; writes test files + checks.yaml |
| `qa.md` | Validates implementation against spec contracts | **KEEP** | Core T6 quality; always-spawn; creates fix tickets for debugger |
| `debugger.md` | Root-cause analysis, produces fix tickets | **KEEP** | T6 swarm agent; conditional but spawned by orchestrator; `/debug:investigate` is user-invoked equivalent |
| `code-reviewer.md` | PR quality gate ensuring spec compliance | **KEEP** | Core T6 quality gate; always-spawn |
| `security-auditor.md` | Independent security review of implementations | **KEEP** | T6 swarm agent; conditional; `/security:review` is user-invoked equivalent |
| `compliance-auditor.md` | Independent compliance review | **KEEP** | T6 swarm agent; conditional; no user-invoked equivalent yet |
| `sdd-specialist.md` | Creates/audits SDD workflow config files | **CONVERT TO SKILL** | User-invoked only; NOT in swarm; 7 modes map to skill group; never referenced by other agents |
| `claude-code-specialist.md` | Creates/optimizes CLAUDE.md, AGENTS.md, agents, skills | **REMOVE (merge)** | User-invoked only; NOT in swarm; 4 modes overlap with sdd-specialist; unique knowledge (Claude Code mechanics) can be absorbed into merged skill group |

---

## Conversion Details

### 1. `sdd-specialist.md` → CONVERT TO SKILL GROUP `/sdd:*`

**Why it qualifies:**
- Performs stateless tasks (no multi-step orchestration across tiers)
- Not referenced by any other agent or workflow (`swarm-config.md` explicitly excludes it)
- Each of its 7 modes is a self-contained operation
- Could be replaced by prompt templates with clear instructions
- No unique tool access (Read, Write, Edit, Glob, Grep — same as skills)

**Proposed skill commands:**

| Current Mode | Proposed Skill | Description |
|-------------|---------------|-------------|
| Mode 1: Create agent file | `/sdd:create-agent` | Create properly structured agent .md with frontmatter |
| Mode 2: Audit agent files | `/sdd:audit-agents` | Check for missing reads/writes, overlapping responsibilities, governance gaps |
| Mode 3: Create/update instructions.md | `/sdd:instructions` | Update SDD workflow instructions for current agent roster |
| Mode 4: Create/update AGENTS.md | `/sdd:agents-md` | Build agent coordination reference |
| Mode 5: Create/update swarm-config.md | `/sdd:swarm-config` | Update tier DAG, auto-detection triggers, roster table |
| Mode 6: Audit SDD workflow files | `/sdd:audit-workflow` | Check consistency across instructions.md, swarm-config.md, AGENTS.md, agents |
| Mode 7: Create skills | `/sdd:create-skill` | Build SDD-related skill files with proper frontmatter |

**Functionality lost in conversion:** None. The agent's context (reading samples, referencing governance reports) can be encoded directly into each skill's instructions. Skills have access to the same tools.

---

### 2. `claude-code-specialist.md` → REMOVE (merge into sdd-specialist skill group)

**Why it qualifies for removal:**
- User-invoked only; NOT in swarm
- **Significant overlap** with sdd-specialist:

| Capability | sdd-specialist | claude-code-specialist |
|-----------|---------------|----------------------|
| Create agent files | Mode 1 | Mode 3 |
| Create skills | Mode 7 | Mode 4 |
| Create/update AGENTS.md | Mode 4 | Mode 2 |
| Audit agents | Mode 2 | — |
| Create/update CLAUDE.md | — | Mode 1 |
| Create/update swarm-config | Mode 5 | — |
| Audit SDD workflow | Mode 6 | — |

**Unique knowledge to preserve:** Claude Code mechanics (CLAUDE.md auto-loading, token budget, skill descriptions vs bodies, agent context isolation, `@` imports). This knowledge should be embedded in the merged skill group's instructions.

**Proposed merge:**
- Absorb Mode 1 (Write CLAUDE.md) into `/sdd:claude-md` skill
- Absorb Mode 2 (Write AGENTS.md) into existing `/sdd:agents-md` skill
- Absorb Mode 3 (Create Agent Files) into existing `/sdd:create-agent` skill — add Claude Code mechanics section
- Absorb Mode 4 (Create Skills) into existing `/sdd:create-skill` skill — add skill frontmatter/description guidance
- Embed Claude Code mechanics as a reference section in the `/sdd:create-agent` and `/sdd:create-skill` skills

**Functionality lost:** None — all capabilities preserved across the skill group with richer instructions.

---

## Existing Skill/Command Overlap Map

Several swarm agents already have user-invoked skill/command equivalents. These are NOT redundancies — the agent form is for swarm orchestration, the skill form is for direct user invocation.

| Swarm Agent | Existing Skill/Command Equivalent | Relationship |
|------------|----------------------------------|-------------|
| `database-administrator` | `/db:plan` + `db-migration` skill | Agent = swarm-compatible; skill = user-invoked |
| `debugger` | `/debug:investigate` + `debugging` skill | Agent = swarm-compatible; skill = user-invoked |
| `security-auditor` | `/security:review` + `security-patterns` skill | Agent = swarm-compatible; skill = user-invoked |
| `qa` | `/spec:validate` (partial) | Skill validates spec completeness; agent validates implementation |
| `ui-designer` | `ux-states` skill (reference only) | Skill = reference patterns; agent = produces ui.md artifact |
| `compliance-engineer` | `compliance-patterns` skill (reference only) | Skill = reference patterns; agent = produces compliance.yaml |

**No existing equivalent (potential skill additions):**
- `compliance-auditor` — No user-invoked skill for compliance review (consider creating `/compliance:review`)
- `code-reviewer` — `/clavix:review` exists but is Clavix-specific (different from the swarm code-reviewer role)

---

## Additional Findings

### Finding 1: Meta-Agent Overlap

`sdd-specialist` and `claude-code-specialist` share 3 of their core capabilities (create agents, create skills, update AGENTS.md). Having two separate agents for overlapping meta-configuration work introduces:
- **Ambiguity**: User must choose which agent to invoke for shared tasks
- **Inconsistency risk**: Two agents may produce different formatting/conventions for the same artifact type
- **Token waste**: Two large agent prompts (~67 lines + ~121 lines) for overlapping roles

### Finding 2: Agent Line Count

| Agent | Lines | Assessment |
|-------|-------|-----------|
| `workflow-orchestrator.md` | 117 | Over 100-line guideline (justified — orchestration complexity) |
| `claude-code-specialist.md` | 121 | Over 100-line guideline (candidate for removal resolves this) |
| `frontend-designer.md` | 99 | At limit |
| `database-administrator.md` | 94 | Good |
| `compliance-auditor.md` | 89 | Good |
| `security-auditor.md` | 84 | Good |
| All others | < 84 | Good |

### Finding 3: Swarm Agents Should NOT Be Converted

All 17 swarm agents are referenced in `swarm-config.md` and spawned by the orchestrator via the Task tool. Converting any swarm agent to a skill would break the orchestration pipeline because:
- Skills run in the main conversation context (not isolated)
- Skills cannot be spawned by the orchestrator as teammates
- Skills don't return summaries to the orchestrator for gate checking
- Skills don't produce `spec-change-requests.yaml` in the swarm protocol

---

## Implementation Checklist

If proceeding with the conversion:

### Phase 1: Create skill group `/sdd:*`
- [ ] Create `.claude/commands/sdd/create-agent.md` (from sdd-specialist Mode 1 + claude-code-specialist Mode 3)
- [ ] Create `.claude/commands/sdd/audit-agents.md` (from sdd-specialist Mode 2)
- [ ] Create `.claude/commands/sdd/instructions.md` (from sdd-specialist Mode 3)
- [ ] Create `.claude/commands/sdd/agents-md.md` (from sdd-specialist Mode 4 + claude-code-specialist Mode 2)
- [ ] Create `.claude/commands/sdd/swarm-config.md` (from sdd-specialist Mode 5)
- [ ] Create `.claude/commands/sdd/audit-workflow.md` (from sdd-specialist Mode 6)
- [ ] Create `.claude/commands/sdd/create-skill.md` (from sdd-specialist Mode 7 + claude-code-specialist Mode 4)
- [ ] Create `.claude/commands/sdd/claude-md.md` (from claude-code-specialist Mode 1)

### Phase 2: Embed Claude Code mechanics
- [ ] Add Claude Code mechanics reference (auto-loading, token budget, skill structure, agent isolation) to `/sdd:create-agent` and `/sdd:create-skill`

### Phase 3: Remove agent files
- [ ] Delete `claude/agents/sdd-specialist.md`
- [ ] Delete `claude/agents/claude-code-specialist.md`
- [ ] Update `AGENTS.md` to remove both from the roster (mark as "converted to /sdd:* skills")
- [ ] Verify `swarm-config.md` does NOT reference either (confirmed: neither is in the roster)

### Phase 4: Verify
- [ ] Run `/sdd:audit-agents` to verify remaining agents are consistent
- [ ] Run `/sdd:audit-workflow` to verify instructions.md/swarm-config.md/AGENTS.md consistency

---

## Summary

| Verdict | Count | Agents |
|---------|-------|--------|
| **KEEP** | 17 | All swarm agents (T1-T6) |
| **CONVERT TO SKILL** | 1 | `sdd-specialist` → `/sdd:*` skill group (8 commands) |
| **REMOVE (merge)** | 1 | `claude-code-specialist` → merged into `/sdd:*` skill group |

**Net result**: 19 agents → 17 agents + 8 new skill commands. No loss of functionality. Clearer separation between swarm agents (orchestrator-managed) and meta-configuration tools (user-invoked skills).

---

**End of Report**

*Cross-references: `claude/agents/swarm-config.md`, `.ops/analysis/agent-governance-report.md`*
