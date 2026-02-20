# Agent Governance Report: Information Flow & Enforcement Gaps

**Generated**: 2026-02-05
**Updated**: 2026-02-06
**Scope**: Analysis of how `product-vision-strategy.md` and `prd.md` content flows through the SDD agent swarm
**Cross-Reference**: `.ops/analysis/vision-document-efficiency-report.md` (2026-02-06)

---

## Executive Summary

### Previous State (2026-02-05)

**Critical Finding**: The SDD agent swarm has a **systematic compliance and security blind spot**. The `product-vision-strategy.md` file contains 15 critical compliance/security definitions (HIPAA/PHIPA readiness, SOC2 baseline, data residency, fail-closed AI, policy-bound execution, zero trust, etc.), but **ZERO agents explicitly reference this file** in their system prompts. These definitions exist only in CLAUDE.md (a distilled subset) and are not consistently propagated into downstream artifacts (specs, system design, tasks).

**Token Efficiency Paradox**: CLAUDE.md explicitly says "Do NOT read `.ops/product-vision-strategy.md` unless needed" (line 83), yet critical agents (compliance-engineer, security-engineer, architect) **need** these definitions to do their jobs correctly.

### Updated State (2026-02-06)

**Resolution**: The `/vision:distill` command (`.claude/commands/vision/distill.md`) now splits the monolithic `product-vision-strategy.md` (~4,400 tokens) into 3 domain-scoped distilled files under `.ops/`:

| Distilled File | Sections | Tokens (approx) | Agent Domain |
|---|---|---|---|
| `quick-product-vision-strategy.md` | §1, §2, §3, §4, §5, §6, §7, §12 | ~1,540 | Product context (spec-writer) |
| `security-compliance-baseline.md` | §10 (partial: compliance & data flow subsections), §11, §12, §15, §16 | ~1,100 | Security/compliance agents |
| `tech-architecture-baseline.md` | §8, §9, §10 (full), §12, §13, §14, §15, §16 | ~2,060 | Architecture agents |

**Token Efficiency Paradox Resolved**: Agents now read domain-scoped files (~1,100-2,060 tokens each) instead of the full document (~4,400 tokens). This represents a **55-75% per-agent token reduction** while closing the enforcement gap. The "Do NOT read" rule in CLAUDE.md can now be replaced with a pointer to the 3 distilled files.

**Remaining Work**: Agent read-contracts (the `## Reads` section in each agent file) still need to be updated to reference these distilled files. The files exist; the agent wiring does not yet.

**Impact** (unchanged):
- Security-critical constraints (e.g., "no silent writes," "Canada-only data residency," "fail-closed AI") may not reach implementation
- Compliance requirements (HIPAA/PHIPA from day 0, SOC2 baseline) lack enforcement checkpoints in the agent pipeline
- PRD constraints (e.g., line 144: "Data residency: Canada only") have no corresponding validation gate

---

## 1. Current State Matrix

| Agent | Currently Reads Vision | Currently Reads PRD | Should Read (Distilled File) | Should Read PRD | Gap Analysis |
|-------|------------------------|---------------------|-----------------------------|-----------------| -------------|
| **Spec Writer** | No | Yes (line 10) | `.ops/quick-product-vision-strategy.md` + `.ops/security-compliance-baseline.md` | Yes | **CRITICAL GAP**: Needs non-goals (§7) from product file + compliance posture, AI boundaries, data constraints (§11, §12) from security file to write correct requirements |
| **Architect** | No | Yes (line 10) | `.ops/tech-architecture-baseline.md` | Yes | **CRITICAL GAP**: Needs architectural approach (§9), data principles (§10), integration philosophy (§13), scalability intent (§14), tech non-goals (§15) to maintain system-design.yaml correctly |
| **Project Task Planner** | No | No | N/A | Yes | **MEDIUM GAP**: Should read PRD to understand build scope context when generating tasks |
| **UI Designer** | No | No | N/A | No | No gap (UI-scoped) |
| **Frontend Designer** | No | No | N/A | No | No gap (component-scoped) |
| **Fullstack Developer** | No | No | N/A | No | No gap (implements from specs/tasks only) |
| **Database Administrator** | No | No | `.ops/tech-architecture-baseline.md` (§10, §14) | No | **MEDIUM GAP**: Needs data principles (§10), scalability intent (§14) for migration strategy decisions |
| **QA** | No | No | N/A | No | No gap (validates against specs) |
| **Test Automator** | No | No | N/A | No | No gap (converts specs to tests) |
| **Debugger** | No | No | N/A | No | No gap (root-cause analysis) |
| **Code Reviewer** | No | No | N/A | No | No gap (reviews against specs/security.md) |
| **Security Engineer** | No | No | `.ops/security-compliance-baseline.md` | Yes | **CRITICAL GAP**: Needs security posture (§11), AI strategy (§12), tech non-goals (§15) to define secure-by-design patterns correctly |
| **Security Auditor** | No | No | `.ops/security-compliance-baseline.md` | Yes | **CRITICAL GAP**: Needs security assumptions (§11), compliance commitments, AI boundaries (§12) to audit against correct baseline |
| **Compliance Engineer** | No | No | `.ops/security-compliance-baseline.md` | Yes | **CRITICAL GAP**: Needs compliance posture (§11), data compliance guarantees (§10 partial), AI strategy (§12), tech non-goals (§15) to translate compliance correctly |
| **Compliance Auditor** | No | No | `.ops/security-compliance-baseline.md` | Yes | **CRITICAL GAP**: Needs compliance commitments (§11: HIPAA/PHIPA, SOC2, GDPR), data residency (§11), audit requirements to verify correctly |
| **Workflow Orchestrator** | No | Yes (line 10) | N/A | Yes | No gap (orchestration-scoped) |
| **Context Manager** | No | No | N/A | No | No gap (records decisions, doesn't need vision) |

### Evidence

**Currently Reads Vision**: Searched all agent files for `product-vision-strategy.md` or `.ops/product-vision-strategy.md` -- **0 matches**

**Currently Reads PRD**:
- `spec-writer.md` line 10: `- '.ops/build/v{x}/prd.md' (primary input)`
- `architect.md` line 10: `- '.ops/build/v{x}/prd.md' (build scope)`
- `workflow-orchestrator.md` line 10: `- '.ops/build/v{x}/prd.md' (build scope)`

**Should Read Vision (Updated Rationale)**:
- **Spec Writer**: Must encode compliance/security/AI constraints into feature requirements. Now reads `quick-product-vision-strategy.md` for non-goals (§7) and `security-compliance-baseline.md` for compliance constraints (§11, §12).
- **Architect**: Must align system-design.yaml with architectural approach, scalability intent, integration philosophy. Now reads `tech-architecture-baseline.md` which contains §8, §9, §10, §12, §13, §14, §15, §16.
- **Security Engineer**: Must reference security assumptions, AI boundaries, and tech non-goals to define patterns. Now reads `security-compliance-baseline.md` which contains §10 partial, §11, §12, §15, §16.
- **Security Auditor**: Must audit against the security posture baseline. Now reads `security-compliance-baseline.md`.
- **Compliance Engineer**: Must translate compliance commitments (HIPAA/PHIPA/SOC2/GDPR) and data principles into technical requirements. Now reads `security-compliance-baseline.md`.
- **Compliance Auditor**: Must verify against compliance commitments and data residency rules. Now reads `security-compliance-baseline.md`.
- **Database Administrator**: Must apply data principles and scalability intent to migration strategies. Now reads `tech-architecture-baseline.md` (§10, §14 specifically).

**Should Read PRD (Rationale)** (unchanged):
- **Project Task Planner**: Currently reads specs only (line 10); should also read PRD for build-level context (constraints, integration points, success metrics)

---

## 2. Information Loss Map

### Critical Definitions from `product-vision-strategy.md` That Don't Reach Downstream Artifacts

> **Updated 2026-02-06**: Each section below is re-evaluated against the distilled files created by `/vision:distill`. Definitions previously marked as "LOST" now have propagation paths via domain-scoped files -- but only once agent read-contracts are updated to reference them.

#### A. Compliance Posture (§11: Security & Compliance Posture)

**Definition Source**: Lines 219-262

**Critical Rules**:
1. "Zero trust by default: Every request is authenticated, authorized, and policy-checked" (line 223)
2. "All actions are attributable: Every read/write/AI action has an actor, purpose, policy decision, and trace ID" (line 225)
3. "Immutable audit log for security-relevant events" (line 236)
4. "Field-level encryption for PHI/PII" (line 244)
5. "Region pinning per tenant (e.g., Canada for PHIPA)" (line 248)
6. "HIPAA/PHIPA readiness: BAAs where applicable" (line 254)
7. "SOC 2 Type II as baseline" (line 255)

**Previous State (2026-02-05)**:
- LOST -- No agent reads vision to extract these
- Partial coverage in CLAUDE.md (lines 8-17: security non-negotiables), but only 3 rules present (no PHI logging, never weaken auth, Supabase RLS)
- PRD mentions "HIPAA/PHIPA from day 0" (line 142) and "Data residency: Canada only" (line 144), but no agent validates these during implementation

**Updated State (2026-02-06)**:
- **RECOVERABLE** -- All 7 rules now exist in `.ops/security-compliance-baseline.md` (which includes full §11)
- Propagation path: `security-compliance-baseline.md` --> Compliance Engineer, Compliance Auditor, Security Engineer, Security Auditor (once read-contracts are updated)
- **Status**: Distilled file exists. Agent wiring pending (see Changes 1.1-1.4).

---

#### B. AI Strategy & Boundaries (§12: AI Strategy)

**Definition Source**: Lines 264-302

**Critical Rules**:
1. "Policy-bound execution: AI can only read/write what the policy engine allows" (line 271)
2. "No silent actions: AI never sends, files, bills, shares, or deletes without explicit user confirmation" (line 273)
3. "No training on customer PHI" (line 274)
4. "Fail-closed: If policy, identity, or context is unclear, AI refuses or asks" (line 275)
5. "Clinical notes finalization (sign-off required)" (line 279)
6. "AI explicitly cannot: Diagnose, prescribe, change policies/permissions, bypass access controls" (line 286-290)

**Previous State (2026-02-05)**:
- LOST -- No agent reads vision to extract these
- CLAUDE.md has no AI-specific rules
- PRD has no AI-specific constraints (v0 doesn't use AI features, but baseline should exist)

**Updated State (2026-02-06)**:
- **RECOVERABLE** -- All 6 rules now exist in `.ops/security-compliance-baseline.md` (includes full §12) AND `.ops/quick-product-vision-strategy.md` (includes §12) AND `.ops/tech-architecture-baseline.md` (includes §12)
- §12 is intentionally included in all 3 distilled files because AI boundaries span all domains
- Propagation path: `security-compliance-baseline.md` --> Security Engineer, Compliance Engineer (once read-contracts are updated); `quick-product-vision-strategy.md` --> Spec Writer (once read-contracts are updated)
- **Status**: Distilled files exist. Agent wiring pending (see Changes 1.1-1.4, 2.1).

---

#### C. Data Principles (§10: Data Principles)

**Definition Source**: Lines 188-216

**Critical Rules**:
1. "No silent writes: Curio never modifies Workspace data without an explicit user/system action + audit entry" (line 203)
2. "Every read/write across boundaries is policy-gated, least-privilege, and logged" (line 213)
3. "Curio maintains provenance: which Workspace objects influenced a note/AI output and under which policy" (line 214)
4. "Prefer minimize PHI duplication; when duplicated, it must be encrypted, scoped, time-bounded, and revocable" (line 215)
5. "Identity/auth/groups: Google Workspace is SoR (Curio mirrors minimal identifiers/roles)" (line 192)
6. "Clinical notes/summaries/AI outputs: Curio is SoR (because governance, retention, and audit must be guaranteed)" (line 194)

**Previous State (2026-02-05)**:
- Partial coverage in CLAUDE.md: "Supabase RLS is non-negotiable" (line 15), but no mention of policy-gated writes, provenance, or PHI minimization
- PRD mentions "No silent writes" (line 146) but only as a constraint reference, not as an enforced pattern

**Updated State (2026-02-06)**:
- **RECOVERABLE** -- Full §10 exists in `.ops/tech-architecture-baseline.md`. Partial §10 (compliance & data flow subsections) exists in `.ops/security-compliance-baseline.md`.
- Propagation paths:
  - `tech-architecture-baseline.md` (full §10) --> Architect, Database Administrator (once read-contracts are updated)
  - `security-compliance-baseline.md` (§10 partial: compliance guarantees + data flow rules) --> Compliance Engineer, Security Engineer (once read-contracts are updated)
- **Status**: Distilled files exist. Agent wiring pending (see Changes 1.1, 1.3, 2.2, 3.1).

---

#### D. Integration Philosophy & Trust Boundaries (§13: Integration Philosophy)

**Definition Source**: Lines 304-337

**Critical Rules**:
1. "External systems are untrusted inputs: Validate, normalize, and policy-check everything coming in" (line 321)
2. "Never delegate compliance to third parties: Curio enforces policy before any outbound write/export" (line 322)
3. "Least-privilege scopes: Minimal OAuth scopes per connector" (line 323)
4. "Policy-gated integration actions: Every read/write is authorized by the policy engine" (line 328)

**Previous State (2026-02-05)**:
- Partial coverage in CLAUDE.md: "Do not relax OAuth scopes, callback URLs, or token handling without spec reference" (line 53)
- LOST: "External systems are untrusted inputs" is not propagated
- LOST: "Policy-gated integration actions" is not propagated

**Updated State (2026-02-06)**:
- **RECOVERABLE** -- Full §13 exists in `.ops/tech-architecture-baseline.md`.
- Propagation path: `tech-architecture-baseline.md` --> Architect (once read-contract is updated)
- **Note**: Security Engineer needs §13 integration trust boundaries but reads `security-compliance-baseline.md`, which does NOT contain §13. However, `security-compliance-baseline.md` does contain §15 (Technical Non-Goals) and §16 (Change Policy), which provide some integration guardrails. For full §13 coverage, Security Engineer could additionally reference `tech-architecture-baseline.md`, or the integration trust boundary rules could be considered covered by the "zero trust" principle in §11 within the security file.
- **Status**: Distilled files exist. Agent wiring pending (see Changes 1.3, 2.2).

---

#### E. Scalability Intent (§14: Scalability Intent)

**Definition Source**: Lines 339-369

**Critical Rules**:
1. "Scale correctness and trust before throughput" (line 342)
2. "Auditability, isolation, and determinism matter more than raw QPS" (line 342)
3. "Per-tenant keys, regional pinning, quotas, and isolation become first-class" (line 366)

**Previous State (2026-02-05)**:
- LOST -- No agent reads vision to extract these

**Updated State (2026-02-06)**:
- **RECOVERABLE** -- Full §14 exists in `.ops/tech-architecture-baseline.md`.
- Propagation path: `tech-architecture-baseline.md` --> Architect, Database Administrator (once read-contracts are updated)
- **Status**: Distilled files exist. Agent wiring pending (see Changes 2.2, 3.1).

---

#### F. Technical Non-Goals (§15: Technical Non-Goals)

**Definition Source**: Lines 371-384

**Critical Constraints**:
1. "No on-prem / self-hosting in the early stages" (line 374)
2. "No offline-first or local PHI storage" (line 378)
3. "No AI autopilot (no unsupervised sending, billing, sharing, deletion, or policy changes)" (line 380)
4. "No storing full duplicates of Workspace content by default" (line 384)

**Previous State (2026-02-05)**:
- LOST -- No agent reads vision to extract these
- PRD does not reference any of these (out-of-scope section is feature-level, not platform-level)

**Updated State (2026-02-06)**:
- **RECOVERABLE** -- Full §15 exists in both `.ops/security-compliance-baseline.md` AND `.ops/tech-architecture-baseline.md` (intentional overlap -- tech non-goals span security and architecture domains).
- Propagation paths:
  - `security-compliance-baseline.md` --> Security Engineer, Compliance Engineer (once read-contracts are updated)
  - `tech-architecture-baseline.md` --> Architect, Database Administrator (once read-contracts are updated)
- Spec Writer needs §15 for feature exclusions. Can access via `quick-product-vision-strategy.md` (which contains §7 non-goals) plus `security-compliance-baseline.md` (which contains §15 tech non-goals).
- **Status**: Distilled files exist. Agent wiring pending (see Changes 1.1, 1.3, 2.1, 2.2, 3.1).

---

### Critical Definitions from `prd.md` That Don't Reach Downstream Artifacts

#### G. PRD Constraints (Section: Constraints, Lines 140-147)

**Definition Source**: PRD lines 140-147

**Critical Constraints**:
1. "HIPAA/PHIPA compliance from day 0" (line 142)
2. "Data residency: Canada only" (line 144)
3. "No silent writes" (line 146)

**Current Propagation** (unchanged from 2026-02-05):
- Spec-writer reads PRD (line 10 in spec-writer.md)
- **No validation gate** -- Compliance-auditor does NOT currently read PRD to check if "Data residency: Canada only" is enforced in implementation
- **No validation gate** -- Security-auditor does NOT currently read PRD to check if "HIPAA/PHIPA from day 0" patterns are applied

**Gap**: Compliance-auditor and security-auditor should read PRD constraints to validate against them. **Currently neither does.**

---

## 3. Enforcement Gap Analysis

### A. Non-Negotiable Rules from Vision WITHOUT Agent Enforcement

| Rule | Source | Should Be Enforced By | Currently Enforced? | Evidence |
|------|--------|----------------------|---------------------|----------|
| "Zero trust by default" | Vision §11, line 223 | Security Engineer (pattern), Security Auditor (check) | No | Neither agent reads vision or distilled files |
| "All actions are attributable" | Vision §11, line 225 | Compliance Engineer (requirement), Compliance Auditor (check) | No | Neither agent reads vision or distilled files |
| "Immutable audit log" | Vision §11, line 236 | Compliance Engineer (requirement), Compliance Auditor (check) | No | Neither agent reads vision or distilled files |
| "Field-level encryption for PHI/PII" | Vision §11, line 244 | Compliance Engineer (requirement), Compliance Auditor (check) | No | Neither agent reads vision or distilled files |
| "Region pinning per tenant (Canada)" | Vision §11, line 248 + PRD line 144 | Compliance Engineer (requirement), Compliance Auditor (check) | No | Neither agent reads vision, distilled files, or PRD constraints |
| "HIPAA/PHIPA readiness" | Vision §11, line 254 + PRD line 142 | Compliance Engineer (requirement), Compliance Auditor (check) | No | Neither agent reads vision, distilled files, or PRD constraints |
| "SOC 2 Type II as baseline" | Vision §11, line 255 | Compliance Engineer (requirement), Compliance Auditor (check) | No | Neither agent reads vision or distilled files |
| "Policy-bound AI execution" | Vision §12, line 271 | Security Engineer (pattern), Compliance Engineer (requirement) | No | Neither agent reads vision or distilled files |
| "No silent AI actions" | Vision §12, line 273 | Security Engineer (pattern), Compliance Engineer (requirement) | No | Neither agent reads vision or distilled files |
| "Fail-closed AI" | Vision §12, line 275 | Security Engineer (pattern), Compliance Engineer (requirement) | No | Neither agent reads vision or distilled files |
| "No training on customer PHI" | Vision §12, line 274 | Compliance Engineer (requirement), Compliance Auditor (check) | No | Neither agent reads vision or distilled files |
| "No silent writes" | Vision §10, line 203 + PRD line 146 | Security Engineer (pattern), Database Administrator (migration gate) | Partial | CLAUDE.md mentions it (implied in Supabase RLS rule), but no agent validates it |
| "Policy-gated writes" | Vision §10, line 213 | Security Engineer (pattern), Code Reviewer (check) | No | Security Engineer doesn't read vision or distilled files |
| "PHI minimization" | Vision §10, line 215 | Compliance Engineer (requirement), Database Administrator (migration check) | No | Neither agent reads vision or distilled files |
| "External systems are untrusted" | Vision §13, line 321 | Security Engineer (pattern) | No | Security Engineer doesn't read vision or distilled files |
| "Never delegate compliance to third parties" | Vision §13, line 322 | Compliance Engineer (requirement) | No | Compliance Engineer doesn't read vision or distilled files |
| "Least-privilege OAuth scopes" | Vision §13, line 323 | Security Engineer (pattern) | Partial | CLAUDE.md line 53: "Do not relax OAuth scopes" but no proactive pattern enforcement |
| "Policy-gated integration actions" | Vision §13, line 328 | Security Engineer (pattern), Security Auditor (check) | No | Neither agent reads vision or distilled files |

**Summary**: **17 critical rules** with **0 full enforcement**, **3 partial enforcement** (via CLAUDE.md distillation).

> **Updated 2026-02-06**: The distilled files (`security-compliance-baseline.md`, `tech-architecture-baseline.md`) now contain all 17 rules in their domain-appropriate files. Once agent read-contracts are updated (Changes 1.1-1.4, 2.1-2.2, 3.1), all 17 rules will have propagation paths to the responsible agents. The "Currently Enforced?" column will change from "No" to "Yes" for rules whose agents read the distilled files.

---

### B. CLAUDE.md Rules vs Vision Content

#### Rules in CLAUDE.md That Originate from Vision

| CLAUDE.md Rule | Vision Source | Coverage Quality |
|----------------|---------------|------------------|
| "NEVER log PHI/PII payloads" (CLAUDE.md line 13) | Vision §11 line 236 (audit requirements) | Good (specific, actionable) |
| "Supabase RLS is non-negotiable" (CLAUDE.md line 15) | Vision §10 line 213 (policy-gated writes) | Partial (narrow -- only Supabase, doesn't cover policy-gating broadly) |
| "Do not relax OAuth scopes" (CLAUDE.md line 53) | Vision §13 line 323 (least-privilege scopes) | Partial (reactive prohibition, no proactive pattern) |

#### Critical Rules from Vision MISSING in CLAUDE.md

| Missing Rule | Vision Source | Impact |
|-------------|---------------|--------|
| "Zero trust by default" | Vision §11 line 223 | HIGH -- Architectural principle not encoded |
| "All actions are attributable" | Vision §11 line 225 | HIGH -- Audit trail not enforced |
| "Field-level encryption for PHI/PII" | Vision §11 line 244 | CRITICAL -- PHI protection gap |
| "Region pinning per tenant" | Vision §11 line 248 | CRITICAL -- Data residency violation risk |
| "HIPAA/PHIPA readiness" | Vision §11 line 254 | CRITICAL -- Compliance gap |
| "SOC 2 Type II as baseline" | Vision §11 line 255 | HIGH -- Security posture gap |
| "Policy-bound AI execution" | Vision §12 line 271 | HIGH -- AI safety gap (v0 doesn't have AI, but baseline needed) |
| "No silent AI actions" | Vision §12 line 273 | HIGH -- AI safety gap |
| "Fail-closed AI" | Vision §12 line 275 | HIGH -- AI safety gap |
| "No training on customer PHI" | Vision §12 line 274 | CRITICAL -- Privacy violation risk |
| "No silent writes" | Vision §10 line 203 | HIGH -- Data integrity gap |
| "PHI minimization" | Vision §10 line 215 | CRITICAL -- PHI over-collection risk |
| "External systems are untrusted" | Vision §13 line 321 | HIGH -- Integration security gap |
| "Policy-gated integration actions" | Vision §13 line 328 | HIGH -- Integration security gap |

**Previous Recommendation** (2026-02-05): CLAUDE.md should either:
1. Include all critical vision rules (bloats token budget), OR
2. Reference vision sections explicitly (e.g., "See `.ops/product-vision-strategy.md` §11 for security posture, §12 for AI strategy")

**Updated Recommendation** (2026-02-06): CLAUDE.md should reference the 3 distilled files as the agent-facing reads, keeping the full vision doc as canonical source for cross-domain analysis only. See Change 4.1.

**Current state**: CLAUDE.md is a **partial distillation** that covers ~20% of critical vision rules, creating a false sense of completeness. The distilled files close this gap by providing domain-complete subsets.

---

### C. PRD Constraint Validation Gaps

| PRD Constraint | Source | Should Be Validated By | Currently Validated? | Evidence |
|----------------|--------|------------------------|----------------------|----------|
| "HIPAA/PHIPA compliance from day 0" | PRD line 142 | Compliance Auditor | No | Compliance Auditor doesn't read PRD |
| "Data residency: Canada only" | PRD line 144 | Compliance Auditor, Database Administrator | No | Neither reads PRD |
| "No silent writes" | PRD line 146 | Security Auditor, Code Reviewer | Partial | Code Reviewer might check if in security.md, but no proactive check |
| "Supabase" (constraint) | PRD line 127 | Architect | Yes | Architect reads PRD (line 10 in architect.md) |
| "WCAG compliant (required, not optional)" | PRD line 112 | QA | No | QA doesn't read PRD, relies on specs.md |

**Recommendation**: Add explicit PRD constraint validation to compliance-auditor and security-auditor workflows.

---

## 4. Recommended Changes (Prioritized by Impact)

> **Updated 2026-02-06**: All read-contract proposals now reference the distilled files created by `/vision:distill` instead of the full `product-vision-strategy.md`. This aligns with the token efficiency analysis in `.ops/analysis/vision-document-efficiency-report.md`.

### Priority 1: CRITICAL -- Compliance & Security Gaps (Immediate Action Required)

#### Change 1.1: Update Compliance Engineer to Read Distilled Vision & PRD

**File**: `claude/agents/compliance-engineer.md`

**Current** (lines 9-13):
```markdown
## Reads
- `.ops/build/v{x}/<feature-name>/specs.md`
- `.ops/build/v{x}/<feature-name>/tasks.yaml`
- Reference `.claude/skills/compliance-patterns/SKILL.md` for data classification, HIPAA safeguards, and compliance templates
- Reference `.claude/skills/sdd-protocols/SKILL.md` for spec-change-requests protocol
```

**Proposed**:
```markdown
## Reads
- `.ops/security-compliance-baseline.md` (§10 partial, §11, §12, §15, §16) -- **REQUIRED**: Extract compliance commitments (HIPAA/PHIPA, SOC2, GDPR), data residency rules, audit requirements, PHI classification, AI boundaries, technical non-goals
- `.ops/build/v{x}/prd.md` (Constraints section) -- **REQUIRED**: Extract build-specific compliance constraints (data residency, compliance scope)
- `.ops/build/v{x}/<feature-name>/specs.md`
- `.ops/build/v{x}/<feature-name>/tasks.yaml`
- Reference `.claude/skills/compliance-patterns/SKILL.md` for data classification, HIPAA safeguards, and compliance templates
- Reference `.claude/skills/sdd-protocols/SKILL.md` for spec-change-requests protocol
```

**Impact**: Ensures compliance requirements from vision (HIPAA/PHIPA, SOC2, data residency) reach `compliance.md` and `specs.md`. Token cost: +1,100 (vs. +4,400 for full vision doc).

---

#### Change 1.2: Update Compliance Auditor to Read Distilled Vision & PRD

**File**: `claude/agents/compliance-auditor.md`

**Current** (lines 9-15):
```markdown
## Reads
- PR diff / changed code
- `.ops/build/v{x}/<feature-name>/compliance.md`
- `.ops/build/v{x}/<feature-name>/specs.md`
- `.ops/build/v{x}/<feature-name>/tasks.yaml`
- Reference `.claude/skills/compliance-patterns/SKILL.md` for data classification and compliance checklist
- Reference `.claude/skills/sdd-protocols/SKILL.md` for checks.yaml merge-only protocol
```

**Proposed**:
```markdown
## Reads
- `.ops/security-compliance-baseline.md` (§11, §12) -- **REQUIRED**: Verify against baseline compliance commitments (HIPAA/PHIPA, SOC2, data residency, audit requirements, AI boundaries)
- `.ops/build/v{x}/prd.md` (Constraints section) -- **REQUIRED**: Verify against build-specific compliance constraints
- PR diff / changed code
- `.ops/build/v{x}/<feature-name>/compliance.md`
- `.ops/build/v{x}/<feature-name>/specs.md`
- `.ops/build/v{x}/<feature-name>/tasks.yaml`
- Reference `.claude/skills/compliance-patterns/SKILL.md` for data classification and compliance checklist
- Reference `.claude/skills/sdd-protocols/SKILL.md` for checks.yaml merge-only protocol
```

**Impact**: Creates enforcement checkpoint for "Data residency: Canada only," "HIPAA/PHIPA from day 0," and other compliance constraints. Token cost: +1,100.

---

#### Change 1.3: Update Security Engineer to Read Distilled Vision & PRD

**File**: `claude/agents/security-engineer.md`

**Current** (lines 9-13):
```markdown
## Reads
- `.ops/build/v{x}/<feature-name>/specs.md`
- `.ops/build/v{x}/<feature-name>/tasks.yaml`
- Reference `.claude/skills/security-patterns/SKILL.md` for auth patterns, OWASP checklist, and threat modeling
- Reference `.claude/skills/sdd-protocols/SKILL.md` for spec-change-requests and checks.yaml protocols
```

**Proposed**:
```markdown
## Reads
- `.ops/security-compliance-baseline.md` (§11, §12, §15) -- **REQUIRED**: Extract security assumptions (zero trust, least privilege, fail-closed), AI boundaries (policy-bound, no silent actions), technical non-goals (no AI autopilot, no local PHI)
- `.ops/build/v{x}/prd.md` (Constraints section) -- **REQUIRED**: Extract build-specific security constraints
- `.ops/build/v{x}/<feature-name>/specs.md`
- `.ops/build/v{x}/<feature-name>/tasks.yaml`
- Reference `.claude/skills/security-patterns/SKILL.md` for auth patterns, OWASP checklist, and threat modeling
- Reference `.claude/skills/sdd-protocols/SKILL.md` for spec-change-requests and checks.yaml protocols
```

**Impact**: Ensures security patterns (`security.md`) encode zero trust, fail-closed AI, policy-gated writes, and technical non-goals. Token cost: +1,100.

---

#### Change 1.4: Update Security Auditor to Read Distilled Vision & PRD

**File**: `claude/agents/security-auditor.md`

**Current** (lines 9-15):
```markdown
## Reads
- PR diff / changed code
- `.ops/build/v{x}/<feature-name>/security.md`
- Runtime config
- Dependency list
- Reference `.claude/skills/security-patterns/SKILL.md` for OWASP checklist and stack-specific security patterns
- Reference `.claude/skills/sdd-protocols/SKILL.md` for checks.yaml merge-only protocol
```

**Proposed**:
```markdown
## Reads
- `.ops/security-compliance-baseline.md` (§11, §12) -- **REQUIRED**: Verify against baseline security assumptions (zero trust, attributable actions, fail-closed AI)
- `.ops/build/v{x}/prd.md` (Constraints section) -- **REQUIRED**: Verify against build-specific security constraints
- PR diff / changed code
- `.ops/build/v{x}/<feature-name>/security.md`
- Runtime config
- Dependency list
- Reference `.claude/skills/security-patterns/SKILL.md` for OWASP checklist and stack-specific security patterns
- Reference `.claude/skills/sdd-protocols/SKILL.md` for checks.yaml merge-only protocol
```

**Impact**: Creates enforcement checkpoint for "Zero trust," "All actions attributable," "Fail-closed AI," and other security baselines. Token cost: +1,100.

---

### Priority 2: HIGH -- Spec & Architecture Alignment (Prevents Drift)

#### Change 2.1: Update Spec Writer to Read Distilled Vision

**File**: `claude/agents/spec-writer.md`

**Current** (lines 9-13):
```markdown
## Reads
- `.ops/build/v{x}/prd.md` (primary input)
- `.ops/build/v{x}/<feature-name>/spec-change-requests.yaml` (if present)
- `.ops/build/system-design.yaml` (if present, for alignment)
- Reference `.claude/skills/sdd-protocols/SKILL.md` for spec-change-requests format and artifact chain
```

**Proposed**:
```markdown
## Reads
- `.ops/quick-product-vision-strategy.md` (§7: Non-Goals) -- **REQUIRED**: Extract product non-goals to encode into feature exclusions
- `.ops/security-compliance-baseline.md` (§11, §12: Compliance & AI constraints) -- **REQUIRED**: Extract compliance constraints, AI boundaries to encode into requirements and non-goals
- `.ops/build/v{x}/prd.md` (primary input)
- `.ops/build/v{x}/<feature-name>/spec-change-requests.yaml` (if present)
- `.ops/build/system-design.yaml` (if present, for alignment)
- Reference `.claude/skills/sdd-protocols/SKILL.md` for spec-change-requests format and artifact chain
```

**Impact**: Ensures non-goals from vision (§7, §15) reach specs.md. Ensures compliance/security constraints reach requirements. Token cost: +1,540 (product) + ~1,100 (security) = ~2,640 total, but spec-writer can section-read for ~1,000 effective tokens.

---

#### Change 2.2: Update Architect to Read Distilled Vision

**File**: `claude/agents/architect.md`

**Current** (lines 9-13):
```markdown
## Reads
- `.ops/build/v{x}/prd.md` (build scope)
- `.ops/build/v{x}/*/specs.md` (skim headings/AC only)
- `.ops/build/system-design.yaml` (if present)
- Reference `.claude/skills/sdd-protocols/SKILL.md` for artifact chain and spec-change-requests format
```

**Proposed**:
```markdown
## Reads
- `.ops/tech-architecture-baseline.md` (§8, §9, §10, §12, §13, §14, §15, §16) -- **REQUIRED**: Extract architectural philosophy, data flow rules, AI strategy, integration patterns, scalability priorities, technical non-goals, and change policy to align system-design.yaml
- `.ops/build/v{x}/prd.md` (build scope)
- `.ops/build/v{x}/*/specs.md` (skim headings/AC only)
- `.ops/build/system-design.yaml` (if present)
- Reference `.claude/skills/sdd-protocols/SKILL.md` for artifact chain and spec-change-requests format
```

**Impact**: Ensures system-design.yaml encodes vision architectural approach (modular monolith, policy engine, event-driven) and rejects designs that violate technical non-goals. Token cost: +2,060.

---

### Priority 3: MEDIUM -- Operational Consistency

#### Change 3.1: Update Database Administrator to Read Distilled Vision

**File**: `claude/agents/database-administrator.md`

**Current** (lines 9-14):
```markdown
## Reads
- `.ops/build/v{x}/<feature-name>/specs.md` (schema intent)
- `.ops/build/v{x}/<feature-name>/tasks.yaml` (DB-related tickets)
- Current DB schema (Supabase dashboard or migration files)
- Reference `.claude/skills/db-migration/SKILL.md` for expand/contract patterns and risk assessment
- Reference `.claude/skills/sdd-protocols/SKILL.md` for checks.yaml and spec-change-requests protocols
```

**Proposed**:
```markdown
## Reads
- `.ops/tech-architecture-baseline.md` (§10: Data Principles, §14: Scalability Intent) -- **RECOMMENDED**: Extract PHI minimization rules, data flow principles, and correctness-over-performance priority for migration strategy
- `.ops/build/v{x}/<feature-name>/specs.md` (schema intent)
- `.ops/build/v{x}/<feature-name>/tasks.yaml` (DB-related tickets)
- Current DB schema (Supabase dashboard or migration files)
- Reference `.claude/skills/db-migration/SKILL.md` for expand/contract patterns and risk assessment
- Reference `.claude/skills/sdd-protocols/SKILL.md` for checks.yaml and spec-change-requests protocols
```

**Impact**: Ensures migration plans respect PHI minimization, provenance tracking, and "scale correctness before throughput." Token cost: +665 (section-specific read from ~2,060 token file).

---

#### Change 3.2: Update Project Task Planner to Read PRD

**File**: `claude/agents/project-task-planner.md`

**Current** (lines 9-12):
```markdown
## Reads
- `.ops/build/v{x}/<feature-name>/specs.md`
- `.ops/build/system-design.yaml` (for architectural context)
- Reference `.claude/skills/sdd-protocols/SKILL.md` for artifact prerequisite chain and spec-change-requests protocol
```

**Proposed**:
```markdown
## Reads
- `.ops/build/v{x}/prd.md` (build scope, constraints) -- **RECOMMENDED**: Extract build-level context (success metrics, integration points, constraints) for task planning
- `.ops/build/v{x}/<feature-name>/specs.md`
- `.ops/build/system-design.yaml` (for architectural context)
- Reference `.claude/skills/sdd-protocols/SKILL.md` for artifact prerequisite chain and spec-change-requests protocol
```

**Impact**: Provides task planner with build-level context to better prioritize and scope tasks.

---

### Priority 4: LOW -- Token Efficiency & Context Distillation

#### Change 4.1: Replace CLAUDE.md "Do NOT read" Rule with Distilled File Pointers

**File**: `CLAUDE.md`

**Current** (line 83):
```markdown
- Do NOT read `.ops/product-vision-strategy.md` or `.ops/ui-design-system.md` unless needed
```

**Proposed**:
```markdown
- `.ops/product-vision-strategy.md` -- Full vision document (~4,400 tokens). Do NOT load unless doing cross-domain analysis or quarterly review.
  - Domain splits (preferred for agents -- generated by `/vision:distill`):
    - `.ops/quick-product-vision-strategy.md` -- Product vision, pillars, non-goals, AI strategy (§1-§7, §12)
    - `.ops/security-compliance-baseline.md` -- Security posture, compliance, AI boundaries, tech non-goals (§10 partial, §11, §12, §15, §16)
    - `.ops/tech-architecture-baseline.md` -- Architecture, data, integration, scalability, tech non-goals (§8-§10, §12-§16)
  - Agents should load only their relevant domain split, not the full document
  - To regenerate distilled files: run `/vision:distill`
- Do NOT read `.ops/ui-design-system.md` unless doing UI work
```

**Impact**: Clarifies when to read vision without loading the full document. Points agents to token-efficient domain splits.

---

#### Change 4.2: Create Distilled Vision Subsets per Agent Category -- DONE

> **Status**: COMPLETED (2026-02-06) via `/vision:distill` command at `.claude/commands/vision/distill.md`

**Original proposal** (2026-02-05): Create new files `.ops/build/vision-compliance.md`, `.ops/build/vision-security.md`, `.ops/build/vision-architecture.md`.

**What was implemented**: `/vision:distill` creates 3 domain-scoped files under `.ops/` with actual names:
- `.ops/quick-product-vision-strategy.md` (§1-§7, §12) -- Product context
- `.ops/security-compliance-baseline.md` (§10 partial, §11, §12, §15, §16) -- Security/compliance
- `.ops/tech-architecture-baseline.md` (§8, §9, §10 full, §12, §13, §14, §15, §16) -- Architecture

**Key design decisions in implementation**:
- §12 (AI Strategy) intentionally appears in all 3 files (AI boundaries span all domains)
- §10 partial extraction for security file: only "Compliance & Auditability Guarantees" and "Data Flow Rules" subsections
- §15 and §16 in both security and architecture files (cross-cutting governance)
- Idempotent, non-destructive, verbatim copy from canonical source
- Each file includes source traceability header (`<!-- Source: ... -->`)

---

## 5. Priority Ranking Summary

| Priority | Change | Impact | Effort | Risk if Not Fixed |
|----------|--------|--------|--------|-------------------|
| **P1** | 1.1: Compliance Engineer reads `security-compliance-baseline.md` & PRD | CRITICAL | Low | HIPAA/PHIPA violations, data residency non-compliance, audit trail gaps |
| **P1** | 1.2: Compliance Auditor reads `security-compliance-baseline.md` & PRD | CRITICAL | Low | No enforcement of "Data residency: Canada only," "HIPAA/PHIPA from day 0" |
| **P1** | 1.3: Security Engineer reads `security-compliance-baseline.md` & PRD | CRITICAL | Low | Zero trust not enforced, fail-closed AI not implemented, integration vulnerabilities |
| **P1** | 1.4: Security Auditor reads `security-compliance-baseline.md` & PRD | CRITICAL | Low | No validation of security baselines (zero trust, attributable actions, fail-closed AI) |
| **P2** | 2.1: Spec Writer reads `quick-product-vision-strategy.md` + `security-compliance-baseline.md` | HIGH | Low | Non-goals not propagated, compliance constraints lost in spec breakdown |
| **P2** | 2.2: Architect reads `tech-architecture-baseline.md` | HIGH | Low | System design drifts from architectural approach, violates technical non-goals |
| **P3** | 3.1: Database Administrator reads `tech-architecture-baseline.md` (§10, §14) | MEDIUM | Low | Migration plans violate PHI minimization, provenance not tracked |
| **P3** | 3.2: Project Task Planner reads PRD | MEDIUM | Low | Tasks lack build-level context (constraints, success metrics) |
| **P4** | 4.1: Update CLAUDE.md to reference distilled files & `/vision:distill` | LOW | Low | Token inefficiency confusion |
| ~~P4~~ | ~~4.2: Create distilled vision subsets~~ | ~~LOW~~ | ~~Medium~~ | **DONE** -- Implemented via `/vision:distill` (2026-02-06) |

---

## 6. Implementation Checklist

### Phase 1: Critical Compliance & Security Fixes (Immediate)
- [x] Update `claude/agents/compliance-engineer.md` per Change 1.1 (add `.ops/security-compliance-baseline.md` to Reads) — **DONE** (2026-02-06)
- [x] Update `claude/agents/compliance-auditor.md` per Change 1.2 (add `.ops/security-compliance-baseline.md` to Reads) — **DONE** (2026-02-06)
- [x] Update `claude/agents/security-engineer.md` per Change 1.3 (add `.ops/security-compliance-baseline.md` to Reads) — **DONE** (2026-02-06)
- [x] Update `claude/agents/security-auditor.md` per Change 1.4 (add `.ops/security-compliance-baseline.md` to Reads) — **DONE** (2026-02-06)
- [ ] Test workflow: Run orchestrator on sample feature — verify compliance.md contains HIPAA/PHIPA + data residency requirements
- [ ] Test workflow: Run compliance-auditor on sample PR — verify blockers for missing data residency enforcement

### Phase 2: Spec & Architecture Alignment
- [x] Update `claude/agents/spec-writer.md` per Change 2.1 (add `.ops/quick-product-vision-strategy.md` + `.ops/security-compliance-baseline.md` to Reads) — **DONE** (2026-02-06)
- [x] Update `claude/agents/architect.md` per Change 2.2 (add `.ops/tech-architecture-baseline.md` to Reads) — **DONE** (2026-02-06)
- [ ] Test workflow: Run spec-writer on sample PRD — verify specs.md Non-Goals section includes vision §7 + §15 items
- [ ] Test workflow: Run architect on sample build — verify system-design.yaml encodes vision §9 architectural approach

### Phase 3: Operational Consistency
- [x] Update `claude/agents/database-administrator.md` per Change 3.1 (add `.ops/tech-architecture-baseline.md` to Reads) — **DONE** (2026-02-06)
- [x] Update `claude/agents/project-task-planner.md` per Change 3.2 (add PRD to Reads) — **DONE** (2026-02-06)
- [ ] Test workflow: Run database-administrator on PHI-related migration — verify risk assessment mentions PHI minimization

### Phase 4: Token Efficiency
- [x] Update `CLAUDE.md` per Change 4.1 (replace "Do NOT read" with distilled file pointers and `/vision:distill` reference) — **DONE** (2026-02-06)
- [x] ~~Create distilled vision subsets per Change 4.2~~ — **DONE** (2026-02-06): Implemented as `/vision:distill` command at `.claude/commands/vision/distill.md`. Run `/vision:distill` to generate distilled files from canonical source.

### Additional Propagation (2026-02-06)
- [x] Update `AGENTS.md` — SDD Artifacts table and folder structure updated with 3 distilled files — **DONE** (2026-02-06)
- [x] Update `claude/agents/instructions.md` — Folder structure updated with 3 distilled files — **DONE** (2026-02-06)

### Prerequisite: Generate Distilled Files
- [x] Run `/vision:distill` to generate the 3 distilled files from `.ops/product-vision-strategy.md` (required before agent read-contracts can take effect)

---

## Appendix A: File Reference Index

### Files Analyzed
1. `.clavix/outputs/prompts/std-20260205-120000-g7x2.md` -- Prompt with analysis instructions
2. `samples/product-vision-strategy.md` -- Vision document (427 lines, 15 critical definition sections)
3. `samples/prd.md` -- PRD document (227 lines, constraints in lines 140-147)
4. `CLAUDE.md` -- Project rules (86 lines, partial vision distillation)
5. `AGENTS.md` -- Agent coordination (199 lines, roster + workflow)
6. `claude/agents/spec-writer.md` -- Planning agent (67 lines)
7. `claude/agents/architect.md` -- Planning agent (66 lines)
8. `claude/agents/project-task-planner.md` -- Planning agent (65 lines)
9. `claude/agents/ui-designer.md` -- Design agent (82 lines)
10. `claude/agents/frontend-designer.md` -- Design agent (98 lines)
11. `claude/agents/fullstack-developer.md` -- Implementation agent (50 lines)
12. `claude/agents/database-administrator.md` -- Implementation agent (92 lines)
13. `claude/agents/qa.md` -- Quality agent (81 lines)
14. `claude/agents/test-automator.md` -- Quality agent (77 lines)
15. `claude/agents/debugger.md` -- Quality agent (66 lines)
16. `claude/agents/code-reviewer.md` -- Quality agent (99 lines)
17. `claude/agents/security-engineer.md` -- Security agent (74 lines)
18. `claude/agents/security-auditor.md` -- Security agent (77 lines)
19. `claude/agents/compliance-engineer.md` -- Compliance agent (84 lines)
20. `claude/agents/compliance-auditor.md` -- Compliance agent (82 lines)
21. `claude/agents/workflow-orchestrator.md` -- Orchestration agent (94 lines)
22. `claude/agents/context-manager.md` -- Orchestration agent (77 lines)
23. `samples/agents/developer.md` -- Sample agent (quality standard reference)
24. `samples/agents/workflow-orchestrator.md` -- Sample agent (keyword-list anti-pattern reference)

### Files Added in Update (2026-02-06)
25. `.claude/commands/vision/distill.md` -- `/vision:distill` command definition
26. `.ops/analysis/vision-document-efficiency-report.md` -- Token efficiency analysis informing distillation
27. `.ops/quick-product-vision-strategy.md` -- Distilled: Product vision (§1-§7, §12) ~1,540 tokens
28. `.ops/security-compliance-baseline.md` -- Distilled: Security/compliance (§10 partial, §11, §12, §15, §16) ~1,100 tokens
29. `.ops/tech-architecture-baseline.md` -- Distilled: Architecture (§8, §9, §10, §12, §13, §14, §15, §16) ~2,060 tokens

### Grep Evidence
- **Vision references in agents**: `grep -r "product-vision-strategy.md" claude/agents/` -- 0 results
- **Distilled file references in agents**: `grep -r "security-compliance-baseline\|tech-architecture-baseline\|quick-product-vision-strategy" claude/agents/` -- 0 results (agent wiring pending)
- **PRD references in agents**: Found in spec-writer.md (line 10), architect.md (line 10), workflow-orchestrator.md (line 10)
- **CLAUDE.md vision reference**: Line 83: "Do NOT read `.ops/product-vision-strategy.md` unless needed"

---

## Appendix B: Critical Vision Sections Quick Reference

### Agent-to-Distilled-File Mapping

| Agent | Distilled File(s) | Key Content Accessed |
|-------|-------------------|---------------------|
| Compliance Engineer | `.ops/security-compliance-baseline.md` | §10 partial (compliance guarantees), §11 (HIPAA/PHIPA, SOC2, data residency, audit), §12 (AI boundaries), §15 (tech non-goals) |
| Compliance Auditor | `.ops/security-compliance-baseline.md` | §11 (compliance commitments, audit requirements), §12 (AI boundaries) |
| Security Engineer | `.ops/security-compliance-baseline.md` | §11 (zero trust, attributable actions), §12 (fail-closed AI, policy-bound), §15 (no AI autopilot) |
| Security Auditor | `.ops/security-compliance-baseline.md` | §11 (security posture baseline), §12 (AI strategy) |
| Spec Writer | `.ops/quick-product-vision-strategy.md` + `.ops/security-compliance-baseline.md` | §7 (non-goals) from product file + §11, §12 (compliance/AI constraints) from security file |
| Architect | `.ops/tech-architecture-baseline.md` | §8 (platform), §9 (arch approach), §10 (data principles), §12 (AI), §13 (integration), §14 (scalability), §15 (tech non-goals) |
| Database Administrator | `.ops/tech-architecture-baseline.md` | §10 (PHI minimization, data flow), §14 (correctness > throughput) |

### Distilled File Traceability

| Distilled File | Source Sections | Source File | Tokens | Regeneration Command |
|---|---|---|---|---|
| `.ops/quick-product-vision-strategy.md` | §1, §2, §3, §4, §5, §6, §7, §12 | `.ops/product-vision-strategy.md` | ~1,540 | `/vision:distill` |
| `.ops/security-compliance-baseline.md` | §10 (partial), §11, §12, §15, §16 | `.ops/product-vision-strategy.md` | ~1,100 | `/vision:distill` |
| `.ops/tech-architecture-baseline.md` | §8, §9, §10 (full), §12, §13, §14, §15, §16 | `.ops/product-vision-strategy.md` | ~2,060 | `/vision:distill` |

**Note on intentional overlaps**:
- §12 (AI Strategy) appears in all 3 distilled files -- AI boundaries span product, security, and architecture domains
- §15 (Technical Non-Goals) appears in security + architecture files -- tech constraints span both domains
- §16 (Change Policy) appears in security + architecture files -- governance spans both domains
- §10 partial (compliance subsections only) in security file; §10 full in architecture file -- compliance agents need guarantees, architecture agents need full data flow rules

### Section-Level Quick Reference (from original report, preserved for traceability)

| Section | Lines | Key Content | Distilled File(s) |
|---------|-------|-------------|-------------------|
| §1-§7: Product Strategy | 1-130 | Vision, problem, customers, value prop, pillars, metrics, non-goals | `quick-product-vision-strategy.md` |
| §8: Core Platform & Tooling | 133-157 | Next.js, Supabase, Stripe, Google Workspace, Vercel | `tech-architecture-baseline.md` |
| §9: Architectural Approach | 160-185 | Modular monolith, API-first, policy engine, event-driven, hybrid compute | `tech-architecture-baseline.md` |
| §10: Data Principles | 188-216 | Source of truth, no silent writes, policy-gated writes, PHI minimization, provenance | `tech-architecture-baseline.md` (full), `security-compliance-baseline.md` (partial) |
| §11: Security & Compliance Posture | 219-262 | Zero trust, attributable actions, audit log, encryption, data residency, HIPAA/PHIPA, SOC2 | `security-compliance-baseline.md` |
| §12: AI Strategy | 264-302 | Policy-bound, no silent actions, fail-closed, no PHI training, human oversight | All 3 distilled files |
| §13: Integration Philosophy | 304-337 | Untrusted inputs, policy-gated actions, least-privilege OAuth, connector certification | `tech-architecture-baseline.md` |
| §14: Scalability Intent | 339-369 | Correctness > throughput, per-tenant isolation, regional pinning | `tech-architecture-baseline.md` |
| §15: Technical Non-Goals | 371-384 | No on-prem, no offline PHI, no AI autopilot, no Workspace duplicates | `security-compliance-baseline.md`, `tech-architecture-baseline.md` |
| §16: Change Policy | 387-421 | What's stable vs. evolving, review cadence, revision triggers | `security-compliance-baseline.md`, `tech-architecture-baseline.md` |

---

**End of Report**

*Original report generated: 2026-02-05*
*Updated: 2026-02-06 -- Reflects `/vision:distill` implementation and distilled file architecture*
*Cross-referenced with: `.ops/analysis/vision-document-efficiency-report.md` (2026-02-06)*
