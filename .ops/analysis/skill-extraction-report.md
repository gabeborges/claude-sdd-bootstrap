# Skill Extraction Analysis Report

**Source prompt**: `comp-20260205-121500-sk2l`
**Agent**: claude-code-specialist
**Date**: 2026-02-05
**Audit method**: Agent-first, then pattern-first

---

## Summary

| Category | Count |
|----------|-------|
| Total agents analyzed | 17 (15 role agents + swarm-config + instructions) |
| Extraction candidates | 9 |
| Extract as skill | 4 |
| Extract as command | 2 |
| Extract as skill + command | 3 |

---

## Part 1: Agent-First Audit

### Agents with extractable behaviors

| Agent | Extractable behavior | Why |
|-------|---------------------|-----|
| spec-writer | Spec output format + validation rules | Users write/validate specs outside orchestration |
| architect | system-design.yaml schema | Users need the schema reference when editing manually |
| project-task-planner | `implements:` pointer validation | Traceability check useful standalone |
| database-administrator | Expand/contract migration planning + risk assessment | Users plan migrations independently of swarm |
| debugger | Root-cause analysis methodology | Users debug outside orchestration constantly |
| security-engineer | Threat modeling patterns + security acceptance checks | Security review is a common standalone activity |
| security-auditor | OWASP checklist + audit methodology | Pairs with security-engineer's patterns |
| compliance-engineer | Data classification framework + compliance templates | Compliance review standalone |
| compliance-auditor | Compliance audit checklist | Pairs with compliance-engineer |

### Agents with NO extractable behaviors (orchestration-only or pure execution)

| Agent | Reason |
|-------|--------|
| workflow-orchestrator | Entire purpose is orchestration routing — not useful standalone |
| context-manager | Sole writer to shared files during orchestration — batch invocation model |
| fullstack-developer | Pure execution role — implements tasks, no reusable knowledge beyond coding |
| qa | Core validation tightly coupled to spec/task artifacts from prior tiers |
| test-automator | Test writing tightly coupled to spec acceptance criteria |
| code-reviewer | Already covered by `/clavix:review` command; spec-compliance logic is orchestration-dependent |
| ui-designer | UX flow documentation too context-dependent to be a useful standalone skill |
| frontend-designer | Already covered by existing `frontend-design` skill + `interface-design` skill |
| claude-code-specialist | Meta-agent for Claude Code config — already standalone by nature |

---

## Part 2: Pattern-First Audit

### Cross-cutting patterns found

| Pattern | Agents affected | Frequency |
|---------|----------------|-----------|
| spec-change-requests.yaml creation | spec-writer, architect, project-task-planner, fullstack-developer, ui-designer, frontend-designer, qa, debugger, code-reviewer, security-engineer, compliance-engineer, security-auditor, compliance-auditor | 13 agents |
| checks.yaml section writing | database-administrator, qa, test-automator, code-reviewer, security-auditor, compliance-auditor | 6 agents |
| `implements:` traceability | project-task-planner, fullstack-developer, qa, code-reviewer, debugger | 5 agents |
| Security knowledge (auth, OWASP, input validation) | security-engineer, security-auditor, code-reviewer | 3 agents |
| Compliance knowledge (PHI/PII, audit trails) | compliance-engineer, compliance-auditor | 2 agents |
| DB migration patterns (expand/contract, rollback) | database-administrator, fullstack-developer | 2 agents |
| Gate/prerequisite checking | workflow-orchestrator, project-task-planner | 2 agents |

---

## Extraction Candidates (ordered by impact)

### 1. Security Patterns

| Field | Value |
|-------|-------|
| **Source agent(s)** | security-engineer, security-auditor, code-reviewer |
| **Extract as** | `skill + command` |
| **Proposed name** | Skill: `security-patterns` (`.claude/skills/security-patterns/SKILL.md`) / Command: `/security:review` |
| **What it does** | Reusable security knowledge: auth patterns, input validation, OWASP Top 10 checklist, secrets management, threat modeling framework |
| **Why extract it** | Security review is the most common standalone activity outside orchestration. Currently security knowledge is split across 3 agents. Users review PRs for security without running the swarm. The skill provides the knowledge; the command provides a user-triggered security audit workflow |
| **SKILL.md sketch** | - Auth/authz patterns (JWT, session, OAuth) with this project's stack (Supabase, Google OAuth) / - OWASP Top 10 checklist tailored to Next.js + Supabase / - Input validation patterns / - Secrets management rules / - `references/owasp-checklist.md` with specific checks |
| **Standalone?** | **Yes** — security review of a PR or file needs no prior tier artifacts |

### 2. DB Migration Planning

| Field | Value |
|-------|-------|
| **Source agent(s)** | database-administrator |
| **Extract as** | `skill + command` |
| **Proposed name** | Skill: `db-migration` (`.claude/skills/db-migration/SKILL.md`) / Command: `/db:plan` |
| **What it does** | Expand/contract migration planning knowledge: pattern templates, risk assessment framework, rollback requirements, `db-migration-plan.yaml` schema |
| **Why extract it** | Users frequently plan DB migrations outside the swarm (e.g., during feature design or when troubleshooting schema issues). The database-administrator agent is 90 lines of domain knowledge that is valuable standalone. Extracting the knowledge into a skill makes the agent lighter and the knowledge reusable by any agent or user |
| **SKILL.md sketch** | - Expand/contract pattern with examples / - Risk assessment matrix (data volume, downtime, rollback complexity) / - `db-migration-plan.yaml` schema reference / - Supabase-specific migration considerations / - `references/patterns.md` with expand/contract/backfill examples |
| **Standalone?** | **Yes** — migration planning only needs the current schema + desired change |

### 3. Spec Validation & Traceability

| Field | Value |
|-------|-------|
| **Source agent(s)** | project-task-planner, qa, code-reviewer, fullstack-developer, debugger |
| **Extract as** | `command` |
| **Proposed name** | `/spec:validate` |
| **What it does** | Validates a feature workspace for completeness: checks specs.md exists and has AC, tasks.yaml has valid `implements:` pointers, every spec node is covered by at least one task, every task traces to a spec node |
| **Why extract it** | Traceability validation appears in 5 agents. Users often want a quick "is this feature workspace complete?" check without running the full swarm. This is the most-requested pre-flight check. Also useful as a CI gate |
| **SKILL.md sketch** | N/A (command only — logic is procedural validation, not domain knowledge) |
| **Standalone?** | **Yes** — only needs the feature workspace files |

### 4. Compliance Patterns

| Field | Value |
|-------|-------|
| **Source agent(s)** | compliance-engineer, compliance-auditor |
| **Extract as** | `skill` |
| **Proposed name** | Skill: `compliance-patterns` (`.claude/skills/compliance-patterns/SKILL.md`) |
| **What it does** | Compliance knowledge: PHI/PII data classification framework, HIPAA/PIPEDA technical controls, audit trail requirements, encryption standards, compliance.md template |
| **Why extract it** | Compliance knowledge is specialized and rarely changes. Extracting it into a skill lets both agents (engineer + auditor) reference the same source of truth. Users can also reference it when designing data handling without running the swarm |
| **SKILL.md sketch** | - Data classification framework (PHI, PII, public) with handling rules / - HIPAA technical safeguards checklist / - Audit trail requirements template / - `references/data-classification.md` with field-level examples |
| **Standalone?** | **Partial** — useful as a reference, but compliance review still needs feature context |

### 5. Gate Readiness Check

| Field | Value |
|-------|-------|
| **Source agent(s)** | workflow-orchestrator, project-task-planner |
| **Extract as** | `command` |
| **Proposed name** | `/gate:check` |
| **What it does** | Pre-flight check for a feature workspace: verifies prd.md exists, specs.md exists and is non-empty, system-design.yaml exists and is up-to-date, tasks.yaml exists with valid pointers. Reports which tier the feature is ready for |
| **Why extract it** | Gate logic currently lives in the orchestrator and is only checked during `/orchestrate`. Users need to know "where am I?" and "what's missing?" before triggering a full swarm run. This saves failed orchestration attempts |
| **SKILL.md sketch** | N/A (command only — procedural checks against the SDD artifact flow) |
| **Standalone?** | **Yes** — reads feature workspace files only |

### 6. Spec-Change-Request Protocol

| Field | Value |
|-------|-------|
| **Source agent(s)** | spec-writer, architect, project-task-planner, fullstack-developer, ui-designer, frontend-designer, qa, debugger, code-reviewer, security-engineer, compliance-engineer, security-auditor, compliance-auditor |
| **Extract as** | `skill` |
| **Proposed name** | Skill: `sdd-protocols` (`.claude/skills/sdd-protocols/SKILL.md`) |
| **What it does** | SDD workflow protocols: spec-change-request format and creation rules, checks.yaml merge-only protocol and section schema, escalation rules, artifact prerequisite chain |
| **Why extract it** | 13 agents reference spec-change-requests.yaml creation, and 6 agents write to checks.yaml. The format and protocol rules are duplicated (implicitly) across every agent. A single skill provides the canonical reference. Agents become lighter because they can reference the skill instead of embedding the protocol inline |
| **SKILL.md sketch** | - spec-change-requests.yaml format + when to create / - checks.yaml section schema and merge-only rules / - Artifact prerequisite chain (prd -> specs -> system-design -> tasks) / - Escalation decision tree |
| **Standalone?** | **Partial** — useful as reference for any agent, but not user-triggered directly |

### 7. Debug Workflow

| Field | Value |
|-------|-------|
| **Source agent(s)** | debugger |
| **Extract as** | `skill + command` |
| **Proposed name** | Skill: `debugging` (`.claude/skills/debugging/SKILL.md`) / Command: `/debug` |
| **What it does** | Root-cause analysis methodology: systematic failure investigation, minimal repro creation, fix ticket format, regression risk assessment |
| **Why extract it** | Debugging is the most common standalone activity — users debug constantly outside orchestration. The debugger agent's methodology (trace, reproduce, narrow, ticket) is valuable as reusable knowledge. The command provides a structured entry point for "help me debug this failure" |
| **SKILL.md sketch** | - Root-cause analysis methodology (reproduce -> trace -> narrow -> verify) / - Fix ticket format template / - Regression risk assessment checklist / - `references/patterns.md` with common failure patterns |
| **Standalone?** | **Yes** — debugging only needs failing test/error + code access |

### 8. Checks Gate Report

| Field | Value |
|-------|-------|
| **Source agent(s)** | database-administrator, qa, test-automator, code-reviewer, security-auditor, compliance-auditor |
| **Extract as** | `command` |
| **Proposed name** | `/gate:report` |
| **What it does** | Reads `checks.yaml` for a feature and produces a unified gate status report across all sections (db_migration, qa_validation, testing, code_review, security_audit, compliance_audit). Shows what passed, what failed, what's missing |
| **Why extract it** | 6 agents write sections to checks.yaml but no single agent reads the whole file to give a unified view. Users need a "show me the gate status" command without running the full orchestration |
| **SKILL.md sketch** | N/A (command only — reads and aggregates checks.yaml) |
| **Standalone?** | **Yes** — only needs the feature checks.yaml file |

### 9. UX State Patterns

| Field | Value |
|-------|-------|
| **Source agent(s)** | ui-designer, frontend-designer |
| **Extract as** | `skill` |
| **Proposed name** | Skill: `ux-states` (`.claude/skills/ux-states/SKILL.md`) |
| **What it does** | UX state enumeration patterns: loading/empty/populated/error states, accessibility requirements (ARIA roles, keyboard nav, focus management), screen state documentation format |
| **Why extract it** | State enumeration and accessibility patterns are referenced by both design agents and are useful during any UI development — even outside orchestration. Could enhance the existing `interface-design` skill, but is distinct enough (behavioral states vs visual craft) to be separate |
| **SKILL.md sketch** | - State enumeration framework (loading, empty, populated, error + edge states) / - Accessibility checklist (ARIA, keyboard nav, focus, contrast) / - Screen state documentation template |
| **Standalone?** | **Partial** — useful as enhancement to UI work, but not independently triggerable |

---

## Agent Simplification After Extraction

| Agent | What gets lighter | How |
|-------|-------------------|-----|
| database-administrator | Inline migration patterns (~40 lines) | References `db-migration` skill instead |
| security-engineer | Inline pattern templates (~30 lines) | References `security-patterns` skill instead |
| security-auditor | OWASP checklist knowledge (~20 lines) | References `security-patterns` skill instead |
| compliance-engineer | Data classification + templates (~35 lines) | References `compliance-patterns` skill instead |
| compliance-auditor | Compliance checklist knowledge (~20 lines) | References `compliance-patterns` skill instead |
| debugger | Methodology section (~25 lines) | References `debugging` skill instead |
| All 13 agents with spec-change-requests | Implicit protocol knowledge | References `sdd-protocols` skill instead |
| 6 agents with checks.yaml | Section format knowledge | References `sdd-protocols` skill instead |

**Estimated total reduction**: ~170 lines removed from agents, consolidated into 6 skills and 4 commands. Each agent becomes focused on its role logic rather than embedding shared knowledge inline.

---

## Recommended Implementation Order

| Priority | Extraction | Rationale |
|----------|-----------|-----------|
| 1 | `sdd-protocols` skill | Highest cross-cutting impact (13 agents); foundation for other extractions |
| 2 | `/spec:validate` command | Most requested pre-flight check; high standalone value |
| 3 | `/gate:check` command | Natural companion to spec:validate; saves failed orchestration runs |
| 4 | `security-patterns` skill + `/security:review` command | Most common standalone activity; 3 agents simplified |
| 5 | `db-migration` skill + `/db:plan` command | High standalone value; simplifies heaviest agent |
| 6 | `debugging` skill + `/debug` command | Very common standalone activity |
| 7 | `compliance-patterns` skill | Similar pattern to security; 2 agents simplified |
| 8 | `/gate:report` command | Natural companion to gate:check |
| 9 | `ux-states` skill | Lowest standalone value; consider merging into `interface-design` skill instead |
