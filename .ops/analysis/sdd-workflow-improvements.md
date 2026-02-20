# SDD Workflow Improvement Analysis

**Generated**: 2026-02-06
**Scope**: Database migration timing + SDD enforcement gaps
**Method**: Evidence-based analysis of 11 source files (CLAUDE.md, AGENTS.md, 7 agent definitions, 2 orchestration configs)

---

## 1. DB Migration Timing Analysis

### Current State: Tier 4 (Implementation Phase)

**Evidence:**
- `AGENTS.md` line 113: database-administrator listed as category "implementation" in Tier 4
- `AGENTS.md` lines 154-168: Dependency DAG places database-administrator at Tier 4 (parallel with frontend-designer)
- `claude/agents/database-administrator.md` line 6: `category: implementation` in frontmatter
- `claude/commands/orchestrate.md` lines 79-86: Tier execution order shows database-administrator runs at T4, AFTER T2 (planning) and T3 (design)
- `claude/commands/orchestrate.md` line 84: database-administrator runs in parallel with frontend-designer at T4

**Process Flow:**
1. T2: project-task-planner creates tasks.yaml
2. T3: frontend-designer creates UI specs
3. T4: database-administrator reads tasks.yaml → creates db-migration-plan.yaml
4. T5: fullstack-developer reads db-migration-plan.yaml → implements

### Upstream Dependencies: What DB Migration Needs

**Current Inputs** (`claude/agents/database-administrator.md` lines 9-15):
- specs.md (schema intent)
- tasks.yaml (DB-related tickets)
- current DB schema

**Missing Inputs** (database-administrator does NOT read):
- system-design.yaml — the architect defines `data.entities` and `data.key_constraints` (architect.md lines 41-43) but database-administrator doesn't consume it
- tech-architecture-baseline.md

**Available at Each Tier:**
- **T2 (Planning)**: specs.md ✓, system-design.yaml ✓, current DB schema ✓
- **T3 (Design)**: All T2 inputs + UI specs
- **T4 (Implementation)**: All T3 inputs + tasks.yaml ✓

**Critical Finding**: Database-administrator has all required inputs available at T2 EXCEPT tasks.yaml. However, tasks.yaml is derived FROM system-design.yaml, which SHOULD contain data architecture decisions.

### Downstream Consumers: Who Reads db-migration-plan.yaml

**Primary Consumer:**
- `claude/agents/fullstack-developer.md` line 13: Reads "db-migration-plan.yaml (if present)"
- `fullstack-developer.md` line 23: "Follow db-migration-plan.yaml exactly for any DB changes"

**Safety Net:**
- `fullstack-developer.md` line 43: Escalation rule — "DB changes needed but no db-migration-plan.yaml -> STOP, request database-administrator"
- This is the ONLY enforcement preventing unplanned DB changes

**No Other Consumers:**
- architect does NOT read db-migration-plan.yaml (architect.md lines 9-13)
- project-task-planner does NOT read db-migration-plan.yaml (project-task-planner.md lines 9-13)

### Risk Assessment: Late Planning (Current T4 State)

**R1: Cascading Rework Risk**
- **Evidence**: `claude/agents/database-administrator.md` line 69: "Schema change breaks existing queries -> create spec-change-requests.yaml, STOP"
- **Impact**: If DB migration planning discovers schema conflicts at T4, all of T2 (tasks.yaml) and T3 (UI specs) may need rework
- **Severity**: HIGH — work has already been completed in 2 prior tiers

**R2: Task Decomposition Blind Spot**
- **Evidence**: `claude/agents/project-task-planner.md` lines 9-13 shows project-task-planner does NOT read anything about DB schema
- **Impact**: tasks.yaml may create tickets that are infeasible due to migration constraints (expand/contract patterns, zero-downtime requirements)
- **Severity**: MEDIUM — tasks may need to be split or resequenced

**R3: Architect-DB Administrator Gap**
- **Evidence**:
  - `claude/agents/architect.md` lines 41-43: architect defines `data.entities` and `data.key_constraints` in system-design.yaml
  - `claude/agents/database-administrator.md` lines 9-15: database-administrator does NOT read system-design.yaml
- **Impact**: Database-administrator may propose migrations that conflict with architect's data model decisions, discovered only at T4
- **Severity**: HIGH — architectural misalignment discovered late

**R4: Parallel Execution Risk**
- **Evidence**: `claude/commands/orchestrate.md` line 84: database-administrator runs in parallel with frontend-designer at T4
- **Impact**: Frontend design proceeds without DB migration plan, may assume DB changes are trivial/instant
- **Severity**: MEDIUM — UI specs may not account for phased rollouts or expand/contract patterns

### Risk Assessment: Early Planning (Moving to T2)

**R5: Over-Planning Risk**
- **Scenario**: Not all features need DB changes — planning migrations for non-DB features wastes cycles
- **Mitigation**: Keyword trigger logic already exists (`orchestrate.md` line 53: DB keywords trigger database-administrator)
- **Severity**: LOW — conditional spawning already in place

**R6: Schema Volatility Risk**
- **Scenario**: Specs change after T2, invalidating DB migration plan
- **Evidence**: `claude/agents/instructions.md` line 14 shows canonical flow has "spec feedback" loop, but `AGENTS.md` lines 57-63 prerequisite table does NOT include db-migration-plan.yaml
- **Mitigation**: spec-change-requests.yaml workflow already handles this (CLAUDE.md line 17)
- **Severity**: LOW — existing feedback loop applies

**R7: Incomplete System Design Risk**
- **Scenario**: Database-administrator runs at T2 before architect has finalized system-design.yaml
- **Evidence**: `AGENTS.md` line 55: Canonical order is specs.md → system-design.yaml → tasks.yaml. Architect runs BEFORE project-task-planner (both T2)
- **Mitigation**: Make database-administrator sequential AFTER architect within T2
- **Severity**: MEDIUM — requires tier sequencing refinement

### Recommendation: **MOVE to T2 (Sequential After Architect)**

**Rationale:**
1. **Input Availability**: All required inputs (specs.md, system-design.yaml, current DB schema) are available at T2 after architect completes
2. **Risk Reduction**: Prevents cascading rework (R1, R3) by surfacing DB constraints before task planning
3. **Better Task Decomposition**: project-task-planner can read db-migration-plan.yaml to create migration-aware tickets
4. **Existing Mitigations**: Conditional spawning (R5) and spec-change-requests workflow (R6) already address early planning risks
5. **Sequential Within T2**: Run database-administrator AFTER architect within T2 to eliminate R7

**Proposed Tier 2 Sequence:**
```
T2-1: architect (reads specs.md → writes system-design.yaml)
T2-2: database-administrator (reads specs.md + system-design.yaml → writes db-migration-plan.yaml)
T2-3: project-task-planner (reads specs.md + system-design.yaml + db-migration-plan.yaml → writes tasks.yaml)
```

**Artifact Flow (Updated):**
```
specs.md (spec-writer)
  ↓
system-design.yaml (architect)
  ↓
db-migration-plan.yaml (database-administrator, conditional)
  ↓
tasks.yaml (project-task-planner)
```

### Edge Cases

**EC1: Specs Change After DB Migration Planning**
- **Current Handling**: `database-administrator.md` line 69 creates spec-change-requests.yaml and stops
- **Proposed Handling**: Same — spec-change-requests.yaml workflow triggers re-planning of affected tiers
- **Impact**: Early planning means fewer downstream artifacts to invalidate (only tasks.yaml, not T3/T4 work)

**EC2: Features Without DB Changes**
- **Current Handling**: `orchestrate.md` line 53 uses DB keywords to conditionally spawn database-administrator
- **Proposed Handling**: Keep keyword trigger — database-administrator only spawns if specs.md contains DB keywords
- **Impact**: No change — conditional spawning already prevents over-planning

**EC3: DB Migration Plan Changes During Implementation**
- **Current Handling**: No explicit mechanism — fullstack-developer.md line 23 says "follow db-migration-plan.yaml exactly"
- **Proposed Handling**: Same — implementation deviations trigger spec-change-requests.yaml
- **Impact**: Early planning makes deviations more visible (divergence from a vetted plan vs. late discovery)

**EC4: Interaction with spec-change-requests.yaml**
- **Current Handling**: Database-administrator creates spec-change-requests.yaml if schema breaks queries (line 69)
- **Proposed Handling**: Same, but earlier detection means:
  - Spec refinement happens BEFORE tasks.yaml exists
  - Less rework in downstream tiers (T3 UI design, T4 implementation)
- **Impact**: Positive — earlier feedback loop convergence

---

## 2. SDD Enforcement Gap Analysis

### Gap 1: Missing DB Migration Plan Gate Before Implementation

**Type**: Gate enforcement gap
**Severity**: **P1 (Critical)**

**Evidence:**
- `claude/commands/orchestrate.md` lines 90-94: Gate checking only verifies completed tasks and spec-change-requests.yaml — NO check for db-migration-plan.yaml
- `claude/agents/workflow-orchestrator.md` line 62: T4 includes database-administrator but no explicit gate ensuring completion before T5
- `claude/agents/fullstack-developer.md` line 43: Escalation rule (self-gate) is the ONLY safety net preventing unplanned DB changes

**Impact:**
- Workflow-orchestrator can advance from T4 to T5 even if database-administrator never ran or failed
- Fullstack-developer self-gates, but this is agent-level enforcement, not workflow-level
- If fullstack-developer skips the self-gate check, unplanned DB changes can proceed

**Current State**: Agent-level self-enforcement only
**Required**: Workflow-level hard gate

**Specific Failure Scenario:**
1. Specs contain DB changes (e.g., "add user preferences table")
2. Database-administrator fails at T4 or is never spawned (keyword miss)
3. Orchestrate.md gate check (lines 90-94) passes (only checks tasks completion)
4. T5 fullstack-developer spawns
5. If fullstack-developer skips line 43 escalation check, implements DB change without migration plan

### Gap 2: Architect-DB Administrator Input Disconnect

**Type**: Artifact dependency validation gap
**Severity**: **P1 (Critical)**

**Evidence:**
- `claude/agents/architect.md` lines 41-43: Architect outputs `data.entities` and `data.key_constraints` in system-design.yaml
- `claude/agents/database-administrator.md` lines 9-15: Database-administrator does NOT read system-design.yaml
- `claude/agents/database-administrator.md` line 36: Process starts with "Read tasks.yaml to identify DB-related tasks" — derives schema intent from specs.md + tasks.yaml instead of architect's canonical data model

**Impact:**
- Database-administrator may design migrations that conflict with architect's data model
- Two sources of truth for data entities: architect's system-design.yaml vs. database-administrator's interpretation of specs.md
- Conflicts discovered only when fullstack-developer tries to implement both

**Current State**: No validation that database-administrator consumes architect's data model
**Required**: Database-administrator MUST read system-design.yaml section `data.entities` and `data.key_constraints`

### Gap 3: Project-Task-Planner DB Blindness

**Type**: Artifact dependency validation gap
**Severity**: **P2 (High)**

**Evidence:**
- `claude/agents/project-task-planner.md` lines 9-13: Reads prd.md, specs.md, system-design.yaml — NO mention of DB schema or migration plans
- `claude/agents/project-task-planner.md` line 28: "Make architectural decisions (defer to architect, database-administrator)" — acknowledges DB expertise but doesn't consume its output
- `AGENTS.md` lines 57-63: Prerequisite table does NOT include db-migration-plan.yaml as input to project-task-planner

**Impact:**
- Tasks.yaml may define tickets that ignore migration complexity:
  - Single ticket for schema change that requires expand/contract pattern (should be 2-3 tickets)
  - No tickets for migration rollback procedures
  - No tickets for data backfill after schema change
- Task estimates ignore migration overhead

**Current State**: Project-task-planner unaware of DB migration complexity
**Required**: Project-task-planner reads db-migration-plan.yaml (if present) to create migration-aware tickets

### Gap 4: No db-migration-plan.yaml in Canonical Artifact Flow

**Type**: Orchestration gap
**Severity**: **P1 (Critical)**

**Evidence:**
- `CLAUDE.md` lines 77-79: SDD Artifact Flow only lists specs.md → system-design.yaml → tasks.yaml — db-migration-plan.yaml is missing
- `AGENTS.md` line 55: Canonical order only covers specs.md → system-design.yaml → tasks.yaml — db-migration-plan.yaml is missing
- `claude/agents/instructions.md` line 14: Canonical flow is "spec → system design → tasks → safe execution" — db-migration-plan.yaml is NOT in this sequence
- `AGENTS.md` line 49: Folder structure shows db-migration-plan.yaml as OPTIONAL in feature workspace

**Impact:**
- DB migration planning is treated as optional, not mandatory for DB-touching features
- No documentation of where db-migration-plan.yaml fits in prerequisite chain
- New contributors may skip DB migration planning entirely (no workflow documentation)

**Current State**: db-migration-plan.yaml not integrated into canonical artifact flow
**Required**: Update CLAUDE.md, AGENTS.md, instructions.md to include db-migration-plan.yaml in artifact sequence

### Gap 5: Missing Build-Level State for DB Migration Planning

**Type**: Orchestration gap
**Severity**: **P2 (High)**

**Evidence:**
- `claude/agents/workflow-orchestrator.md` lines 31-34: Build-Level State Machine only tracks specs_updated → system_design_updated → tasks_generated
- No state for `db_migration_planned` or `db_migration_not_needed`

**Impact:**
- Workflow-orchestrator cannot answer "is DB migration planning complete for this build?"
- No visibility into whether database-administrator needs to run or has already run
- State machine cannot enforce db-migration-plan.yaml prerequisite before tasks.yaml creation (proposed T2 sequencing)

**Current State**: No state tracking for DB migration planning
**Required**: Add `db_migration_planned` state to workflow-orchestrator state machine

### Gap 6: Teammate Prompt Template Omits db-migration-plan.yaml

**Type**: Agent boundary violation (read contract enforcement gap)
**Severity**: **P3 (Medium)**

**Evidence:**
- `claude/agents/swarm-config.md` lines 64-85: Teammate prompt template tells agents to read prd.md, specs.md, system-design.yaml, tasks.yaml
- db-migration-plan.yaml is NOT mentioned in the template

**Impact:**
- Agents spawned by orchestrator are not instructed to check for db-migration-plan.yaml
- Fullstack-developer's line 13 "reads db-migration-plan.yaml (if present)" is agent-specific knowledge, not enforced by spawn template
- Other implementation agents (if added) won't know to check for DB migration plan

**Current State**: Agent read contracts not enforced by spawn template
**Required**: Add db-migration-plan.yaml to teammate prompt template (conditional: "if present")

### Gap 7: No Prerequisite Rule for db-migration-plan.yaml

**Type**: Artifact dependency validation gap
**Severity**: **P1 (Critical)**

**Evidence:**
- `AGENTS.md` lines 57-63: Prerequisite rules table covers specs.md, system-design.yaml, tasks.yaml
- NO rule documented for db-migration-plan.yaml

**Prerequisite Rules Table (Current):**

| Artifact | Cannot Start Until | File Reference |
|----------|---------------------|----------------|
| system-design.yaml | specs.md exists | architect.md line 29 |
| tasks.yaml | system-design.yaml complete | project-task-planner.md line 32 |
| *db-migration-plan.yaml* | *MISSING* | *MISSING* |

**Impact:**
- No documented rule for when database-administrator can start
- No documented rule for what must exist before db-migration-plan.yaml creation
- Ambiguity: should database-administrator wait for tasks.yaml (current T4 behavior) or system-design.yaml (proposed T2 behavior)?

**Current State**: No prerequisite rule
**Required**: Add rule "db-migration-plan.yaml cannot start until system-design.yaml complete"

### Gap 8: Spec-Change-Request Feedback Loop Not Enforced for DB Conflicts

**Type**: Feedback loop enforcement gap
**Severity**: **P3 (Medium)**

**Evidence:**
- `claude/agents/database-administrator.md` line 69: "Schema change breaks existing queries -> create spec-change-requests.yaml, STOP"
- `claude/commands/orchestrate.md` lines 90-94: Gate check looks for spec-change-requests.yaml and halts workflow
- BUT: No documented process for what happens AFTER spec-change-requests.yaml is created by database-administrator

**Impact:**
- Database-administrator stops correctly when conflict detected
- But no documented workflow for:
  - Who reviews spec-change-requests.yaml?
  - Who updates specs.md?
  - How does workflow restart after spec refinement?
- Risk of dead-end: spec-change-requests.yaml created, workflow halts, no resolution path

**Current State**: Escalation exists, resolution workflow undocumented
**Required**: Document spec-change-requests.yaml resolution process in AGENTS.md or orchestrate.md

### Gap 9: Agent File Path Discrepancy

**Type**: Documentation inconsistency (not an enforcement gap, but creates confusion)
**Severity**: **P4 (Low)**

**Evidence:**
- `AGENTS.md` lines 105-121: Roster table lists agent files at `.claude/agents/<name>.md` (with leading dot)
- Actual file paths: `claude/agents/<name>.md` (NO leading dot)
- Example: AGENTS.md line 111 says ".claude/agents/database-administrator.md" but actual file is "claude/agents/database-administrator.md"

**Impact:**
- New contributors following AGENTS.md will look in wrong directory
- Scripts/tools expecting `.claude/` prefix will fail
- Confusion about canonical agent location

**Current State**: Documentation path mismatch
**Required**: Fix AGENTS.md lines 105-121 to remove leading dot from file paths

### Gap 10: No Enforcement of "Do Not Create tasks.yaml Until system-design.yaml Exists"

**Type**: Gate enforcement gap
**Severity**: **P2 (High)**

**Evidence:**
- `CLAUDE.md` line 79: "Do not create tasks.yaml until system-design.yaml exists"
- `claude/agents/project-task-planner.md` line 32: Hard Stop Rule checks for system-design.yaml
- BUT: This is agent-level self-enforcement, not workflow-level gate
- `claude/commands/orchestrate.md` lines 90-94: Workflow gate does NOT verify system-design.yaml exists before spawning project-task-planner

**Impact:**
- If project-task-planner is manually invoked (bypassing orchestrator), it can self-gate
- But if orchestrator spawns project-task-planner prematurely, only agent's self-check prevents violation
- No workflow-level enforcement of documented CLAUDE.md rule

**Current State**: Agent-level self-enforcement only
**Required**: Orchestrate.md gate must verify system-design.yaml exists before spawning T2 project-task-planner

---

## 3. Priority Ranking (All Recommendations)

### P1 (Critical) — Must Fix Before Production Use

1. **Gap 1**: Add workflow-level gate for db-migration-plan.yaml before T5 fullstack-developer
2. **Gap 2**: Database-administrator MUST read system-design.yaml data model
3. **Gap 4**: Add db-migration-plan.yaml to canonical artifact flow in CLAUDE.md, AGENTS.md, instructions.md
4. **Gap 7**: Document prerequisite rule for db-migration-plan.yaml creation
5. **DB Timing**: Move database-administrator to T2 (sequential after architect)

### P2 (High) — Significant Risk Reduction

6. **Gap 3**: Project-task-planner reads db-migration-plan.yaml to create migration-aware tickets
7. **Gap 5**: Add db_migration_planned state to workflow-orchestrator state machine
8. **Gap 10**: Orchestrate.md gate enforces system-design.yaml exists before project-task-planner spawns

### P3 (Medium) — Quality & Consistency Improvements

9. **Gap 6**: Add db-migration-plan.yaml to teammate prompt template
10. **Gap 8**: Document spec-change-requests.yaml resolution workflow

### P4 (Low) — Documentation Cleanup

11. **Gap 9**: Fix agent file path discrepancy in AGENTS.md (remove leading dots)

---

## 4. Implementation Checklist (NOT EXECUTED — Reference Only)

### Phase 1: DB Migration Tier Move (P1)

- [ ] **claude/agents/database-administrator.md**
  - [ ] Line 6: Change `category: implementation` to `category: planning`
  - [ ] Lines 9-15 (Reads section): Add `system-design.yaml (data.entities, data.key_constraints)` after specs.md
  - [ ] Line 36: Change process start from "Read tasks.yaml" to "Read system-design.yaml data model, then specs.md schema intent"

- [ ] **AGENTS.md**
  - [ ] Line 113: Move database-administrator from Tier 4 to Tier 2, update category to "planning"
  - [ ] Lines 154-168: Update dependency DAG — database-administrator in T2, sequential after architect, before project-task-planner
  - [ ] Line 55: Update canonical order to "specs.md → system-design.yaml → db-migration-plan.yaml (if DB changes) → tasks.yaml"

- [ ] **claude/commands/orchestrate.md**
  - [ ] Lines 79-86: Move database-administrator from T4 to T2
  - [ ] Add sequencing note: "T2 sequence: architect → database-administrator (if DB keywords) → project-task-planner"

- [ ] **claude/agents/swarm-config.md**
  - [ ] Line 19: Move database-administrator tier from 4 to 2
  - [ ] Lines 35-43: Update Tier DAG to show database-administrator in T2

### Phase 2: Artifact Flow Integration (P1)

- [ ] **CLAUDE.md**
  - [ ] Lines 77-79: Update SDD Artifact Flow to: "specs.md (spec-writer) → system-design.yaml (architect) → db-migration-plan.yaml (database-administrator, conditional) → tasks.yaml (project-task-planner)"
  - [ ] Add line after 79: "Do not create db-migration-plan.yaml until system-design.yaml exists (for DB-touching features only)"

- [ ] **AGENTS.md**
  - [ ] Line 55: Update canonical order (same as CLAUDE.md change)
  - [ ] Lines 57-63: Add row to prerequisite table:
    ```
    | db-migration-plan.yaml | system-design.yaml complete AND specs contain DB keywords | database-administrator.md line 36 |
    ```
  - [ ] Line 49: Change db-migration-plan.yaml from "optional" to "conditional (required if DB changes detected)"

- [ ] **claude/agents/instructions.md**
  - [ ] Line 14: Update canonical flow to include "db migration planning" between system design and tasks
  - [ ] Lines 86-104: Update database-administrator inputs to include system-design.yaml

### Phase 3: Gate Enforcement (P1)

- [ ] **claude/commands/orchestrate.md**
  - [ ] Lines 90-94: Add gate check before T5:
    ```
    if tasks.yaml contains DB changes AND db-migration-plan.yaml missing:
      STOP, output: "DB changes detected in tasks but no migration plan — spawn database-administrator first"
    ```
  - [ ] Add gate check before T2 project-task-planner:
    ```
    if system-design.yaml missing:
      STOP, output: "Cannot create tasks.yaml — system-design.yaml prerequisite missing"
    ```

- [ ] **claude/agents/workflow-orchestrator.md**
  - [ ] Lines 31-34: Add state to build-level state machine:
    ```
    db_migration_planned: bool (true if db-migration-plan.yaml exists OR feature has no DB changes)
    ```
  - [ ] Lines 67-70: Update gate check to verify db_migration_planned == true before T5

### Phase 4: Cross-Agent Integration (P2)

- [ ] **claude/agents/project-task-planner.md**
  - [ ] Lines 9-13 (Reads section): Add `db-migration-plan.yaml (if present — for migration-aware task breakdown)`
  - [ ] Add process step after line 36: "If db-migration-plan.yaml exists, create tickets for each migration phase (expand, migrate, contract)"

- [ ] **claude/agents/architect.md**
  - [ ] Add clarification after line 43: "Output data.entities and data.key_constraints will be consumed by database-administrator for migration planning"

### Phase 5: Template & Documentation (P3)

- [ ] **claude/agents/swarm-config.md**
  - [ ] Lines 64-85: Add to teammate prompt template:
    ```
    - db-migration-plan.yaml (if feature involves DB changes)
    ```

- [ ] **AGENTS.md**
  - [ ] Add new section after line 168: "## Spec Change Request Resolution Workflow"
    ```
    When any agent creates spec-change-requests.yaml:
    1. Workflow-orchestrator halts tier progression
    2. Human or spec-writer reviews requests
    3. Specs.md is updated
    4. Affected downstream artifacts (system-design.yaml, db-migration-plan.yaml, tasks.yaml) are regenerated
    5. Workflow resumes from first affected tier
    ```

### Phase 6: Path Fix (P4)

- [ ] **AGENTS.md**
  - [ ] Lines 105-121: Remove leading dot from all agent file paths
    - Change `.claude/agents/` to `claude/agents/` (11 occurrences in roster table)

---

## 5. Validation Checklist

- [x] All 11 specified source files were read and referenced with line numbers
- [x] DB migration recommendation is one of MOVE / KEEP / HYBRID with clear rationale: **MOVE to T2 (sequential after architect)**
- [x] Each enforcement gap cites specific file and line as evidence (10 gaps identified, all cited)
- [x] Proposed fixes are specific file edits with line numbers (not vague "improve documentation")
- [x] No changes were made to any files — report only (output is this analysis document)

---

## Appendix: Evidence Cross-Reference

### Files Analyzed
1. CLAUDE.md (93 lines)
2. AGENTS.md (205 lines)
3. claude/commands/orchestrate.md (111 lines)
4. claude/agents/instructions.md (138 lines)
5. claude/agents/swarm-config.md (95 lines)
6. claude/agents/database-administrator.md (93 lines)
7. claude/agents/architect.md (67 lines)
8. claude/agents/project-task-planner.md (66 lines)
9. claude/agents/fullstack-developer.md (50 lines)
10. claude/agents/workflow-orchestrator.md (94 lines)
11. claude/agents/spec-writer.md (69 lines)

### Key Evidence Summary

**DB Migration Current State:**
- database-administrator.md line 6 (category: implementation)
- AGENTS.md line 113 (Tier 4 categorization)
- orchestrate.md lines 79-86 (Tier execution order)

**Artifact Flow Gaps:**
- CLAUDE.md lines 77-79 (missing db-migration-plan.yaml)
- AGENTS.md line 55 (missing db-migration-plan.yaml)
- AGENTS.md lines 57-63 (no prerequisite rule for db-migration-plan.yaml)

**Gate Enforcement Gaps:**
- orchestrate.md lines 90-94 (no db-migration-plan.yaml check)
- workflow-orchestrator.md lines 31-34 (no db_migration_planned state)
- workflow-orchestrator.md line 62 (no T4→T5 gate for DB migration)

**Cross-Agent Integration Gaps:**
- database-administrator.md lines 9-15 (doesn't read system-design.yaml)
- architect.md lines 41-43 (outputs data model) vs. database-administrator.md line 36 (doesn't consume it)
- project-task-planner.md lines 9-13 (doesn't read db-migration-plan.yaml)

---

**End of Report**
