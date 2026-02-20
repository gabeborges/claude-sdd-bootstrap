# Token Efficiency Report: SDD Pipeline Execution Architecture

**Generated**: 2026-02-08
**Updated**: 2026-02-08 — v3: slash command modes, agent merge (ui-designer+frontend-designer), conciseness directive
**Scope**: Dynamic token cost analysis of `/orchestrate` agent swarm execution
**Agent performing analysis**: SDD Specialist

---

## Executive Summary

The execution architecture of `/orchestrate` spawns too many agents, re-reads the same artifacts redundantly, produces verbose output, and forces sequential processing where parallelism is possible.

A full `/orchestrate` run for one feature:
- Spawns **up to 16 agents** across 6 sequential tiers
- Makes **~30 round-trips** between orchestrator and agents
- Re-reads `specs.md` **11 times** independently (once per agent)
- Re-reads `tasks.yaml` **11 times** independently
- Keeps the orchestrator's context growing across all tiers (accumulating every agent summary)
- Runs T2 as **4 fully sequential spawns** (the biggest bottleneck)
- Splits UI work across 2 agents + 2 tiers when 1 agent in 1 tier would suffice
- Produces verbose agent summaries that bloat orchestrator context

The token cost is **O(agents × artifacts_per_agent × summary_verbosity)**, not O(total_file_sizes).

---

## 1. Execution Cost Model

### 1.1 What Happens During `/orchestrate`

```
ORCHESTRATOR (persistent, growing context)
│
├─ T1: Spawn context-manager ──────────── 1 spawn, 1 round-trip
│   └─ Returns: "ready"
│   └─ Orchestrator routes summary → context-manager (1 more round-trip)
│
├─ T2: Sequential chain ───────────────── 4 spawns, 8+ round-trips
│   ├─ Spawn spec-writer → waits → receives summary
│   │   └─ Orchestrator routes summary → context-manager
│   ├─ Spawn architect → waits → receives summary
│   │   └─ Orchestrator routes summary → context-manager
│   │   └─ IF spec-change-requests → rerun spec-writer → rerun architect (LOOP)
│   ├─ Spawn db-admin (if DB keywords) → waits → receives summary
│   │   └─ Orchestrator routes summary → context-manager
│   └─ Spawn project-task-planner → waits → receives summary
│       └─ Orchestrator routes summary → context-manager
│
├─ T3: Parallel optionals ─────────────── 0-3 spawns, 0-6 round-trips
│   ├─ Spawn ui-designer (if UI keywords)
│   ├─ Spawn security-engineer (if security keywords)
│   └─ Spawn compliance-engineer (if compliance keywords)
│   └─ Orchestrator routes summaries → context-manager
│
├─ T4: Conditional ────────────────────── 0-1 spawn, 0-2 round-trips
│   └─ Spawn frontend-designer (if ui-designer ran)
│       └─ Orchestrator routes summary → context-manager
│
├─ T5: Parallel builders ──────────────── 2 spawns, 4 round-trips
│   ├─ Spawn fullstack-developer
│   └─ Spawn test-automator
│   └─ Orchestrator routes summaries → context-manager
│
└─ T6: Parallel validation ────────────── 3-5 spawns, 6-10 round-trips
    ├─ Spawn qa (always)
    ├─ Spawn debugger (if QA finds bugs)
    ├─ Spawn code-reviewer (always)
    ├─ Spawn security-auditor (if security-engineer ran)
    └─ Spawn compliance-auditor (if compliance-engineer ran)
    └─ Orchestrator routes summaries → context-manager
```

### 1.2 Spawn Count by Scenario

| Scenario | T1 | T2 | T3 | T4 | T5 | T6 | Total Spawns | Total Round-Trips |
|----------|----|----|----|----|----|----|-------------|------------------|
| **Full build** (all keywords match) | 1 | 4 | 3 | 1 | 2 | 5 | **16** | **~32** |
| **Typical build** (UI + security, no compliance) | 1 | 4 | 2 | 1 | 2 | 4 | **14** | **~28** |
| **Minimal build** (no UI, no security, no compliance) | 1 | 4 | 0 | 0 | 2 | 3 | **10** | **~20** |
| **Iterative build** (specs + tasks exist, just implement) | 1 | 0* | 0 | 0 | 2 | 3 | **6** | **~12** |

*T2 is skipped entirely only if all planning artifacts already exist. Currently the orchestrator always runs T2.

### 1.3 Artifact Re-Reading (The Hidden Multiplier)

Every agent independently reads its required artifacts. No content is shared between agents.

| Artifact | Times Read Per Build | Agents That Read It |
|----------|---------------------|-------------------|
| `specs.md` | **11** | spec-writer, architect, db-admin, planner, ui-designer, security-eng, compliance-eng, frontend-designer, fullstack-dev, test-automator, qa, code-reviewer |
| `tasks.yaml` | **11** | planner, security-eng, compliance-eng, frontend-designer, fullstack-dev, test-automator, qa, code-reviewer, debugger, security-auditor, compliance-auditor |
| `checks.yaml` | **8** (read/write) | db-admin, test-automator, qa, code-reviewer, security-auditor, compliance-auditor, context-manager |
| `prd.md` | **5** | spec-writer, architect, planner, security-auditor, compliance-auditor |
| `security-compliance-baseline.md` | **5** | spec-writer, security-eng, security-auditor, compliance-eng, compliance-auditor |

**For a typical feature (specs.md ~80 lines, tasks.yaml ~50 lines):**
- `specs.md`: 80 lines × 11 reads = **880 lines read redundantly**
- `tasks.yaml`: 50 lines × 11 reads = **550 lines read redundantly**
- Total: **~1,430 lines** (~5,000 tokens) of redundant artifact reading

### 1.4 Orchestrator Context Growth

The orchestrator's context never shrinks — it accumulates:

| After Tier | Accumulated Content | Est. Context Size |
|-----------|--------------------|--------------------|
| Start | System prompt + @ imports + artifact reads | ~1,500 tokens |
| After T1 | + context-manager summary | ~1,800 tokens |
| After T2 | + 4 agent summaries + gate checks | ~3,500 tokens |
| After T3 | + 0-3 agent summaries | ~4,500 tokens |
| After T4 | + 0-1 agent summary | ~5,000 tokens |
| After T5 | + 2 agent summaries (likely large — code output) | ~8,000 tokens |
| After T6 | + 3-5 agent summaries | ~11,000+ tokens |

By T6 the orchestrator carries **~11,000+ tokens** of accumulated context. Verbose agent summaries make this worse — agents tend to narrate their process instead of reporting results.

---

## 2. Root Causes (Ranked by Impact)

### Root Cause 1: No Execution Modes — Full Pipeline Always Runs

**Impact: HIGH — 4-10 unnecessary spawns per iterative build**

The orchestrator always runs the full T1→T6 pipeline regardless of what already exists. If you're iterating on a feature where `specs.md`, `system-design.yaml`, and `tasks.yaml` already exist, T2 still spawns 4 agents to regenerate them.

Most real development is iterative. You run `/orchestrate` once to plan, then again to implement changes. The second run shouldn't re-plan from scratch.

---

### Root Cause 2: T2 Sequential Chain (4 Agents in Series)

**Impact: HIGH — Single biggest time bottleneck**

T2 forces 4 agents to run in strict sequence: spec-writer → architect → db-admin → project-task-planner. Each must fully complete before the next starts. Any delay blocks all downstream agents. If spec-change-requests appear, the loop restarts.

**Why it exists:** Each agent's output is the next agent's input. Genuine data dependency.

**But:** If artifacts already exist and only need incremental updates, the full chain is wasteful.

---

### Root Cause 3: Redundant Artifact Reads

**Impact: MEDIUM — ~5,000 tokens of redundant reading per build**

11 agents independently read `specs.md`. 11 agents independently read `tasks.yaml`. No content is passed between agents — each starts with a blank slate and re-reads everything.

**Why it exists:** Each agent spawns in an isolated context window (by design — Claude Code agents don't share state).

---

### Root Cause 4: Context-Manager Per-Tier Overhead

**Impact: MEDIUM — 6 unnecessary routing cycles for simple builds**

The orchestrator routes agent summaries to context-manager after **every tier**, even when there's nothing to log. For a clean build with no deviations, context-manager processes 6 batches but only writes routine "task completed" entries.

---

### Root Cause 5: ui-designer + frontend-designer Are Two Agents for One Job

**Impact: MEDIUM — 1 extra spawn + 1 extra tier for UI work**

UI work is split across two agents in two tiers:
- `ui-designer` (T3, 82 lines): Defines UX flows, screens, states → writes `ui.md`
- `frontend-designer` (T4, 99 lines): Reads `ui.md`, adds component architecture → updates `ui.md`

`frontend-designer` literally reads what `ui-designer` just wrote and appends to the same file. This is one workflow split into two agents with a tier boundary between them. The handoff loses context — frontend-designer must re-interpret ui-designer's intent from the artifact rather than having it in working memory.

A single agent with both responsibilities would:
- Save 1 spawn + 1 round-trip
- Eliminate an entire tier (T4)
- Have full context of UX intent when defining components (no handoff gap)
- Produce a single, coherent `ui.md` in one pass

---

### Root Cause 6: Engineer/Auditor Duplication

**Impact: LOW-MEDIUM — 2-4 extra spawns for security + compliance**

Security and compliance each have two agents:
- `security-engineer` (T3) defines patterns → `security-auditor` (T6) reviews implementation
- `compliance-engineer` (T3) defines requirements → `compliance-auditor` (T6) reviews implementation

For early-stage builds (v0, v1), the separation adds spawns without proportional value. The "define then audit" independence matters for mature codebases, not early MVPs.

---

### Root Cause 7: Agent Verbosity

**Impact: MEDIUM — Inflates orchestrator context by ~30-50%**

Agents produce verbose summaries: restating the task, narrating their process, adding preambles and disclaimers. The orchestrator accumulates these summaries across tiers. Verbose output from 16 agents compounds into significant context growth.

A 200-word agent summary that could be 50 words wastes ~150 words × 16 agents = **~2,400 unnecessary words** (~3,200 tokens) per full build.

---

## 3. Recommended Changes (Ranked by Impact)

### Change 1: Execution Mode Slash Commands

**Saves: 4-10 spawns per iterative run**

Replace the single `/orchestrate` command with a command group:

```
/orchestrate:plan .ops/build/v0/user-auth      → T1-T2 only (planning)
/orchestrate:build .ops/build/v0/user-auth     → T4-T5 only (implement + validate)
/orchestrate:validate .ops/build/v0/user-auth  → T5 only (QA + review)
/orchestrate .ops/build/v0/user-auth           → auto-detect mode (default)
```

**Why slash commands instead of flags:**
- Consistent with existing patterns (`/clavix:plan`, `/gate:check`, `/security:review`)
- Users see all modes in their slash command list — no flags to memorize
- Each mode has its own description visible in the command palette
- `$ARGUMENTS` stays clean (just the path, no flag parsing)

**Auto-detection logic (for bare `/orchestrate`):**
```
IF specs.md missing OR prd.md newer than specs.md:
  mode = full (T1→T5)
ELIF tasks.yaml missing OR specs.md newer than tasks.yaml:
  mode = plan+build (T2→T5)
ELIF implementation files missing OR tasks have status: pending:
  mode = build (T4→T5)
ELSE:
  mode = validate (T5)
```

**Implementation:**
- Create `claude/commands/orchestrate/` directory
- Move `claude/commands/orchestrate.md` → `claude/commands/orchestrate/auto.md` (or keep as default)
- Create `claude/commands/orchestrate/plan.md` (T1-T2 only)
- Create `claude/commands/orchestrate/build.md` (T4-T5 only)
- Create `claude/commands/orchestrate/validate.md` (T5 only)
- Each command file shares the same base orchestrator logic but restricts tier range
- Update `workflow-orchestrator.md` to accept mode parameter

**Risk:** LOW — All modes use the same orchestrator agent, just with different tier ranges. Full pipeline available via bare `/orchestrate` on a fresh workspace.

---

### Change 2: Artifact Pre-Loading by Orchestrator

**Saves: ~5,000 tokens of redundant reads per build**

The orchestrator reads shared artifacts once and passes content in each agent's spawn prompt.

**Current pattern (in swarm-config.md Teammate Prompt Template):**
```
Read the following artifacts before starting:
1. .ops/build/v{x}/prd.md
2. {feature-path}/specs.md
3. .ops/build/system-design.yaml
```

**Proposed pattern:**
```
## Pre-loaded Artifacts (DO NOT re-read these files)

### specs.md
{actual content, pre-read by orchestrator}

### tasks.yaml
{actual content, pre-read by orchestrator}

Read on-demand only if needed for your role:
- .ops/build/system-design.yaml (architect, db-admin, planner)
- .ops/build/v{x}/prd.md (spec-writer, architect, planner)
```

**Pre-load rule:** Only artifacts read by 5+ agents (`specs.md`, `tasks.yaml`). If either exceeds 150 lines, fall back to on-demand.

**Risk:** MEDIUM — Increases initial prompt size per agent. 150-line threshold prevents oversized prompts.

---

### Change 3: Merge ui-designer + frontend-designer into Single Agent

**Saves: 1 spawn, 1 tier, ~500 tokens orchestrator overhead**

Combine both agents into a single `ui-designer` with expanded scope:

**Current (2 agents, 2 tiers):**
```
T3: ui-designer → writes ui.md (flows, screens, states)
T4: frontend-designer → reads ui.md, adds component architecture
```

**Proposed (1 agent, 1 tier):**
```
T3: ui-designer → writes complete ui.md (flows + screens + states + component architecture)
```

**What the merged agent does:**
1. Define UX flows, screen definitions, interaction states, accessibility (current ui-designer scope)
2. Then translate those into component hierarchy, props, states, data requirements (current frontend-designer scope)
3. Produce a single, coherent `ui.md` with both UX and component architecture

**Why this works better than two agents:**
- No handoff gap — the agent that defined the UX intent also defines components, so nothing is lost in translation
- One artifact write instead of write-then-update
- Full context of design decisions when making component choices
- Both agents already write to the same file (`ui.md`)

**Implementation:**
- Merge `frontend-designer.md` content into `ui-designer.md`:
  - Add component architecture to the Process section (props, states, children, data requirements)
  - Add "Reference existing codebase components for reuse" to Reads
  - Update the output format to include component breakdowns after each flow
- Delete `claude/agents/frontend-designer.md`
- Update `swarm-config.md`: remove frontend-designer row, remove T4 entirely
- Update `instructions.md`: remove frontend-designer from DAG and Agent I/O table
- Update `AGENTS.md`: remove frontend-designer entry

**Risk:** LOW — No capability lost. Same output, one agent instead of two.

---

### Change 4: Lazy Context-Manager (Event-Driven, Not Per-Tier)

**Saves: 3-5 routing cycles for clean builds (~1,500 tokens)**

Only invoke context-manager when:
1. A routing trigger keyword is detected (deviation, spec break, blocked, etc.)
2. All tiers complete (final summary batch)

**Current:** 6 invocations per full build (once per tier)
**Proposed:** 1-2 invocations per clean build (final + any deviations)

Checkpoint: Always route after T2 (planning decisions are highest-value to preserve).

**Risk:** LOW-MEDIUM — Decisions-log.md written at end instead of incrementally. T2 checkpoint mitigates crash risk.

---

### Change 5: Merge Engineer + Auditor Pairs for Early Builds

**Saves: 2-4 spawns for v0/v1 builds**

For build versions v0/v1, combine:
- `security-engineer` + `security-auditor` → `security-agent`
- `compliance-engineer` + `compliance-auditor` → `compliance-agent`

```
IF build_version <= v1:
  T3: spawn security-agent (define patterns)
  T5: spawn security-agent again (review implementation)
ELSE:
  T3: spawn security-engineer (separate)
  T5: spawn security-auditor (separate)
```

**Risk:** MEDIUM — Same agent defines and audits (self-review). Acceptable for early builds.

---

### Change 6: Smart T2 — Skip Unchanged Artifacts

**Saves: 1-3 spawns when artifacts are current**

Before spawning each T2 agent, check if its output is newer than its input:

```
spec-writer: SKIP if specs.md exists AND newer than prd.md
architect: SKIP if system-design.yaml exists AND newer than specs.md
db-admin: SKIP if db-migration-plan.yaml exists AND current
planner: SKIP if tasks.yaml exists AND newer than specs.md
```

This is implicitly handled by `/orchestrate:build` (which skips T2 entirely), but also useful for bare `/orchestrate` auto-detection.

**Risk:** MEDIUM — Stale artifacts could be wrongly considered fresh. Bare `/orchestrate` on a fresh workspace always runs full pipeline as fallback.

---

### Change 7: Conciseness Directive in Teammate Prompt Template

**Saves: ~3,200 tokens of verbose summaries per full build**

Add a conciseness block to the Teammate Prompt Template in `swarm-config.md`:

```markdown
## Communication Rules

- Summaries: ≤ 5 sentences. Bullets, not paragraphs.
- Findings: structured format (tables, YAML, lists). No prose.
- No filler: skip preamble, restatements, "I'll now proceed to...", "Let me analyze..."
- When talking to the user: answer directly. Do not narrate your process.
- When reporting to orchestrator: state what you did, what you produced, and any blockers. Nothing else.
```

**Why the teammate prompt template:**
- Single injection point — all 15+ agents inherit it automatically
- No need to edit each agent file individually
- Orchestrator builds spawn prompts from this template, so every agent gets the directive

**Risk:** LOW — Agents may occasionally be too terse, but that's preferable to the current verbosity. Users can always ask for more detail.

---

## 4. Impact Summary

### Before: Full Build (16 agents, 6 tiers, ~32 round-trips)

```
T1  ●─────────────────────────────────────────── context-manager
T2  ●────●────●────●──────────────────────────── spec-writer → architect → db-admin → planner
T3  ●════●════●═══════════════════════════════── ui-designer ║ security-eng ║ compliance-eng
T4  ●─────────────────────────────────────────── frontend-designer
T5  ●════●════════════════════════════════════── fullstack-dev ║ test-automator
T6  ●════●════●════●════●═════════════════════── qa ║ debugger ║ reviewer ║ sec-audit ║ comp-audit

Spawns: 16 | Round-trips: ~32 | Tiers: 6
```

### After: Full Build with All Changes (11 agents, 5 tiers, ~20 round-trips)

```
T1  (deferred — lazy context-manager)
T2  ●────●────●────●──────────────────────────── spec-writer → architect → db-admin → planner
T3  ●════●════●═══════════════════════════════── ui-designer ║ security-agent ║ compliance-agent
T4  ●════●════════════════════════════════════── fullstack-dev ║ test-automator
T5  ●════●════●════●══════════════════════════── qa ║ debugger ║ reviewer ║ sec-agent ║ comp-agent

Spawns: 11 | Round-trips: ~20 | Tiers: 5 (was 6)
Artifact pre-loading saves: ~5,000 tokens
Conciseness directive saves: ~3,200 tokens
Context-manager: 1-2 invocations (keyword-triggered + final)
```

### After: Iterative Build via `/orchestrate:build` (5 agents, 2 tiers, ~10 round-trips)

```
T4  ●════●════════════════════════════════════── fullstack-dev ║ test-automator
T5  ●════●════●═══════════════════════════════── qa ║ reviewer ║ sec-agent

Spawns: 5 | Round-trips: ~10 | Tiers: 2
Context-manager: invoked once at end
```

### Savings by Change

| # | Change | Spawns Saved | Tokens Saved | Risk |
|---|--------|-------------|-------------|------|
| 1 | Execution mode slash commands | 4-10 | 3,000-8,000 | LOW |
| 2 | Artifact pre-loading | 0 | ~5,000 | MEDIUM |
| 3 | Merge ui-designer + frontend-designer | 1 | ~800 | LOW |
| 4 | Lazy context-manager | 0 | ~1,500 | LOW-MED |
| 5 | Merge engineer+auditor (v0/v1) | 2-4 | 1,500-3,000 | MEDIUM |
| 6 | Smart T2 skip | 1-3 | 1,000-3,000 | MEDIUM |
| 7 | Conciseness directive | 0 | ~3,200 | LOW |
| | **Total (full build)** | **4-9** | **~16,000** | |
| | **Total (iterative build)** | **9-15** | **~22,500** | |

---

## 5. Implementation Order

### Phase 0: Highest Impact, Lowest Risk

1. **Merge ui-designer + frontend-designer** — merge agent files, update swarm-config, eliminate T4
2. **Conciseness directive** — add communication rules to teammate prompt template in swarm-config.md
3. **Execution mode slash commands** — create `orchestrate/plan.md`, `orchestrate/build.md`, `orchestrate/validate.md`

### Phase 1: Moderate Impact

4. **Artifact pre-loading** — update teammate prompt template to inline specs.md + tasks.yaml
5. **Lazy context-manager** — buffer summaries, route only on keywords or completion

### Phase 2: Requires New Agent Files

6. **Merge engineer+auditor pairs** — create `security-agent.md` and `compliance-agent.md` for v0/v1 builds
7. **Smart T2 skip** — add freshness checks to orchestrator (partially handled by `/orchestrate:build`)

---

## 6. Static File Optimization (Lower Priority)

Secondary to execution architecture changes. For completeness:

| Change | Lines Saved | Tokens Saved | Priority |
|--------|-------------|-------------|----------|
| Extract Clavix Agent Manual to shared skill | ~11,100 | ~38,850 | P3 (irrelevant to /orchestrate) |
| Remove Example sections from agents | ~250 | ~875 | P2 |
| Extract checks.yaml format to sdd-protocols | ~48 | ~168 | P2 |
| Merge AGENTS.md into instructions.md | ~200 | ~700 | P2 |
| Remove/convert sdd-specialist + claude-code-specialist | ~186 | ~651 | P2 |

---

## 7. Agent Teams Evaluation

**Source**: [Claude Code Agent Teams docs](https://code.claude.com/docs/en/agent-teams) (experimental feature)

Agent Teams let you coordinate multiple **full Claude Code sessions** (not Task-tool subagents). Each teammate is an independent Claude Code instance with its own context window, CLAUDE.md loading, MCP servers, and skills.

### Verdict: Not Suitable for Token Reduction

The docs explicitly warn: *"Agent teams use significantly more tokens than a single session."*

Agent Teams solve a **different problem** — collaborative exploration and debate. The SDD pipeline is a **deterministic DAG** with predefined roles. These paradigms don't align.

| Root Cause | Agent Teams Help? | Why |
|---|---|---|
| RC1: No execution modes | No | Don't decide what work to skip |
| RC2: T2 sequential chain | No | Can't parallelize genuine data dependencies |
| RC3: Redundant artifact reads | **Worsens** | Each teammate loads full project context independently |
| RC4: Context-manager overhead | Neutral | Shared task list replaces routing, but messaging adds overhead |
| RC5: UI agent split | No | Doesn't consolidate agents |
| RC6: Engineer/auditor duplication | No | Designed to add teammates, not consolidate |
| RC7: Agent verbosity | No | Each teammate still produces its own summaries |

**Ideas worth borrowing (without adopting Agent Teams):**
1. Shared task list with dependency auto-unblocking — reduces orchestrator round-trips
2. Task self-claiming — agents pick up next unblocked task without orchestrator intervention

**Recommendation:** Do not adopt Agent Teams. Implement Changes 1-7 instead.

---

## 8. Implementation Plan

### Phase 0: Highest Impact, Lowest Risk

#### 0.1 — Merge ui-designer + frontend-designer

**Files to modify:**
- `claude/agents/ui-designer.md` — expand scope to include component architecture
- `claude/agents/swarm-config.md` — remove frontend-designer, eliminate T4, renumber tiers
- `claude/agents/instructions.md` — update DAG and Agent I/O table
- `AGENTS.md` — remove frontend-designer entry

**Delete:**
- `claude/agents/frontend-designer.md`

**Changes to `ui-designer.md`:**

Add to Reads:
```
- Existing codebase components (for reuse)
```

Add to Writes:
```
- Component architecture appended to ui.md (hierarchy, props, states, data requirements)
```

Add to Rules — Must do:
```
- After defining UX flows, produce component architecture for each screen
- Define component hierarchy with props interfaces and state management
- Reference existing components in the codebase for reuse
- Map components back to flow screens
```

Remove from Rules — Must NOT do:
```
- Write implementation code or component definitions (frontend-designer's job)
```
Replace with:
```
- Write implementation code
```

Add to Process, after the flow/screen template:
```markdown
Then for each screen, produce:

## Component: {ComponentName}
**Source flow**: {flow reference}
**Reuses**: {existing component, if any}

### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|

### States
- **Loading**: {what renders}
- **Empty**: {what renders}
- **Populated**: {what renders}
- **Error**: {what renders}

### Children
- {ChildComponent} — {purpose}

### Data Requirements
- {API call or store selector needed}
```

**Changes to `swarm-config.md`:**

Remove frontend-designer row from Agent Roster. Renumber:
```
T1: context-manager
T2: spec-writer → architect → database-administrator → project-task-planner (sequential)
T3: ui-designer ║ security-engineer ║ compliance-engineer (parallel, if detected)
T4: fullstack-developer ║ test-automator (parallel)
T5: qa ║ debugger ║ code-reviewer ║ security-auditor ║ compliance-auditor (parallel)
```

Update auto-detection: UI keywords now trigger only `ui-designer` (no frontend-designer).

**Risk:** LOW — Same output, one agent.

---

#### 0.2 — Conciseness Directive

**Files to modify:**
- `claude/agents/swarm-config.md` — add to Teammate Prompt Template

**Add after the Context section in the Teammate Prompt Template:**

```markdown
## Communication Rules

- Summaries: ≤ 5 sentences. Bullets, not paragraphs.
- Findings: structured format (tables, YAML, lists). No prose.
- No filler: skip preamble, restatements, "I'll now proceed to...", "Let me analyze..."
- When talking to the user: answer directly. Do not narrate your process.
- When reporting to orchestrator: state what you did, what you produced, and any blockers. Nothing else.
```

**Risk:** LOW.

---

#### 0.3 — Execution Mode Slash Commands

**Files to create:**
- `claude/commands/orchestrate/plan.md`
- `claude/commands/orchestrate/build.md`
- `claude/commands/orchestrate/validate.md`

**Files to modify:**
- `claude/commands/orchestrate.md` — update to be the auto-detect default
- `claude/agents/workflow-orchestrator.md` — accept mode parameter

**`orchestrate/plan.md`** — Plan mode (T1-T2 only):
```markdown
---
description: "SDD planning phase only — produce specs, system design, and tasks"
---
# /orchestrate:plan — Planning Only

Same as /orchestrate but restricted to Tiers 1-2 (planning).
Spawn: context-manager, spec-writer, architect, database-administrator (if DB), project-task-planner.
Stop after T2. Do not proceed to design, implementation, or validation.

Feature path: $ARGUMENTS
```

**`orchestrate/build.md`** — Build mode (T3-T5 only):
```markdown
---
description: "SDD implementation + validation — requires existing specs and tasks"
---
# /orchestrate:build — Build Only

Same as /orchestrate but restricted to Tiers 3-5 (design + implementation + validation).
Pre-check: specs.md and tasks.yaml MUST exist. If missing, STOP and tell user to run /orchestrate:plan first.
Spawn: ui-designer (if UI), security/compliance (if detected), fullstack-developer, test-automator, qa, code-reviewer, auditors.

Feature path: $ARGUMENTS
```

**`orchestrate/validate.md`** — Validate mode (T5 only):
```markdown
---
description: "SDD validation only — QA, review, and audit of existing implementation"
---
# /orchestrate:validate — Validation Only

Same as /orchestrate but restricted to Tier 5 (validation).
Pre-check: implementation must exist. If no code changes found, STOP.
Spawn: qa, code-reviewer, debugger (if issues), security-auditor (if security), compliance-auditor (if compliance).

Feature path: $ARGUMENTS
```

**Update `orchestrate.md`** — becomes auto-detect:
- Add auto-detection logic (check artifact freshness)
- Route to appropriate tier range based on workspace state

**Risk:** LOW — Each command shares the same orchestrator logic with restricted tier ranges.

---

### Phase 1: Moderate Impact

#### 1.1 — Artifact Pre-Loading

**Files to modify:**
- `claude/agents/swarm-config.md` — update Teammate Prompt Template

Replace "Read the following artifacts" with pre-loaded content section. Pre-load `specs.md` and `tasks.yaml` if under 150 lines each.

**Risk:** MEDIUM.

#### 1.2 — Lazy Context-Manager

**Files to modify:**
- `claude/agents/workflow-orchestrator.md` — buffer summaries, route only on triggers or completion

**Risk:** LOW-MEDIUM.

---

### Phase 2: Requires New Agent Files

#### 2.1 — Merge Engineer + Auditor Pairs (v0/v1)

**Files to create:**
- `claude/agents/security-agent.md`
- `claude/agents/compliance-agent.md`

**Files to modify:**
- `claude/agents/swarm-config.md` — version-conditional agent selection

**Risk:** MEDIUM.

#### 2.2 — Smart T2 Skip

**Files to modify:**
- `claude/agents/workflow-orchestrator.md` — freshness checks before T2 spawns

Partially handled by `/orchestrate:build` (skips T2 entirely). Useful for bare `/orchestrate` auto-detection.

**Risk:** MEDIUM.

---

### Implementation Priority Matrix

| Phase | Change | Spawns Saved | Tokens Saved | Risk | Effort | Files Changed |
|-------|--------|-------------|-------------|------|--------|--------------|
| **0** | Merge ui+frontend agents | 1 | ~800 | LOW | Low | 4 files + 1 delete |
| **0** | Conciseness directive | 0 | ~3,200 | LOW | Trivial | 1 file |
| **0** | Execution mode commands | 4-10 | 3,000-8,000 | LOW | Medium | 3 new + 2 modified |
| **1** | Artifact pre-loading | 0 | ~5,000 | MEDIUM | Low | 1 file |
| **1** | Lazy context-manager | 0 | ~1,500 | LOW-MED | Low | 1 file |
| **2** | Merge eng+auditor | 2-4 | 1,500-3,000 | MEDIUM | Medium | 2 new + 1 modified |
| **2** | Smart T2 skip | 1-3 | 1,000-3,000 | MEDIUM | Low | 1 file |

---

## 9. New Tier DAG (Post-Implementation)

```
T1: context-manager (lazy — invoked on triggers or at end)
T2: spec-writer → architect → database-administrator → project-task-planner (sequential, skippable)
T3: ui-designer ║ security-engineer ║ compliance-engineer (parallel, if detected)
T4: fullstack-developer ║ test-automator (parallel)
T5: qa ║ debugger ║ code-reviewer ║ security-auditor ║ compliance-auditor (parallel)
```

**Always spawn**: fullstack-developer (T4), test-automator (T4), qa (T5), code-reviewer (T5).

**Slash command tier ranges:**
| Command | Tiers | Use Case |
|---------|-------|----------|
| `/orchestrate:plan` | T1-T2 | Planning only |
| `/orchestrate:build` | T3-T5 | Design + implement + validate |
| `/orchestrate:validate` | T5 | QA + review only |
| `/orchestrate` | Auto-detect | Routes to appropriate range |

---

## Cross-References

- `.ops/analysis/agent-to-skill-conversion-report.md` (2026-02-08) — Agent removal/conversion
- `.ops/analysis/agent-governance-report.md` (2026-02-06) — Agent read-contract gaps
- `claude/agents/swarm-config.md` — Current tier DAG and auto-detection keywords
- `claude/agents/workflow-orchestrator.md` — Orchestrator process definition
- [Claude Code Agent Teams docs](https://code.claude.com/docs/en/agent-teams) — Evaluated and rejected (Section 7)

---

**End of Report**

*v1 (2026-02-08): Static file size analysis*
*v2 (2026-02-08): Execution architecture focus — tier consolidation, spawn reduction, artifact sharing*
*v3 (2026-02-08): Slash command modes, ui-designer+frontend-designer merge, conciseness directive, Agent Teams evaluation*
