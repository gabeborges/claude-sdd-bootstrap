# Vision Document Efficiency Report

**Generated**: 2026-02-06
**Source**: `samples/product-vision-strategy.md` (template for `.ops/product-vision-strategy.md`)
**Cross-Reference**: `.ops/analysis/agent-governance-report.md` (2026-02-05)

---

## Executive Summary

The product-vision-strategy.md is a 427-line, ~3,315-word monolithic document (~4,400 tokens) containing 16 sections spanning product strategy, architecture, security, compliance, AI, data, integration, and scalability. When any agent needs content from this file, it loads the entire document — yet no single agent needs more than 35% of it. The governance report found that 7 agents critically need this document but zero currently read it, creating a compliance/security blind spot. **Decomposing into 4 focused domain documents would reduce per-agent token load by 55–80% while closing the enforcement gap identified in the governance report.**

---

## 1. Current State Analysis

### 1.1 Document Profile

| Metric | Value |
|--------|-------|
| Total lines | 427 |
| Total words | ~3,315 |
| Estimated tokens (1.33x words) | ~4,400 |
| Number of sections | 16 |
| Agents that currently read it | 0 |
| Agents that should read it (per governance report) | 7 (P1–P3) |
| Agents with partial benefit | 5 additional (P3–P4) |

### 1.2 Section-by-Section Token Breakdown

| § | Section | Lines | Words (approx) | Tokens (approx) | Primary Domain |
|---|---------|-------|-----------------|------------------|----------------|
| 1 | Vision | 1–13 | 95 | 125 | Product |
| 2 | Problem Space | 16–31 | 135 | 180 | Product |
| 3 | Target Customers | 33–49 | 115 | 155 | Product |
| 4 | Value Proposition | 52–83 | 280 | 370 | Product |
| 5 | Strategic Pillars | 86–97 | 100 | 130 | Product |
| 6 | Success Metrics | 100–119 | 165 | 220 | Product |
| 7 | Non-Goals | 122–130 | 60 | 80 | Product / Spec Boundary |
| 8 | Core Platform & Tooling | 133–157 | 195 | 260 | Architecture |
| 9 | Architectural Approach | 160–185 | 250 | 330 | Architecture |
| 10 | Data Principles | 188–216 | 260 | 345 | Data / Compliance |
| 11 | Security & Compliance Posture | 219–262 | 370 | 490 | Security / Compliance |
| 12 | AI Strategy | 265–302 | 325 | 430 | AI / Compliance |
| 13 | Integration Philosophy | 306–337 | 280 | 370 | Integration / Security |
| 14 | Scalability Intent | 340–368 | 240 | 320 | Architecture |
| 15 | Technical Non-Goals | 372–384 | 115 | 155 | Architecture / Spec Boundary |
| 16 | Change Policy | 387–421 | 210 | 280 | Governance (cross-cutting) |
| — | Headers/separators | — | ~20 | ~85 | Formatting |
| **Total** | | **427** | **~3,315** | **~4,400** | |

### 1.3 Relevance Ratios by Agent Role

When an agent loads the full document, most content is irrelevant to its task:

| Agent | Sections Needed | Relevant Tokens | % of Total | Waste |
|-------|----------------|-----------------|------------|-------|
| Compliance Engineer | §10, §11, §12, §13 | ~1,635 | 37% | 63% |
| Compliance Auditor | §10, §11, §12 | ~1,265 | 29% | 71% |
| Security Engineer | §11, §12, §13 | ~1,290 | 29% | 71% |
| Security Auditor | §11, §12 | ~920 | 21% | 79% |
| Architect | §8, §9, §10, §13, §14, §15 | ~1,780 | 40% | 60% |
| Spec Writer | §7, §10, §11, §12, §15 | ~1,500 | 34% | 66% |
| Database Administrator | §10, §14 | ~665 | 15% | 85% |

**Average relevance ratio across 7 critical agents: 29%** — meaning 71% of loaded tokens are wasted on average.

---

## 2. Domain Clustering Analysis

### 2.1 Natural Groupings

Sections cluster into 4 clear domains based on content affinity and consumer overlap:

#### Cluster A: Product Strategy (§1–§7)
- **Content**: Vision, problem space, customers, value proposition, pillars, metrics, non-goals
- **Words**: ~950 | **Tokens**: ~1,260
- **Consumers**: Spec Writer (§7), UI Designer (§5), Product stakeholders
- **Update frequency**: Low (quarterly review)
- **Internal cross-references**: High within cluster (pillars reference vision, metrics reference pillars)
- **External dependencies**: None — self-contained strategic context

#### Cluster B: Security, Compliance & AI Boundaries (§11, §12, parts of §10)
- **Content**: Security posture, compliance commitments, AI strategy, data compliance guarantees
- **Words**: ~830 | **Tokens**: ~1,100
- **Consumers**: Security Engineer, Security Auditor, Compliance Engineer, Compliance Auditor
- **Update frequency**: Low (changes only on regulatory shifts or security incidents)
- **Internal cross-references**: §11 ↔ §12 (AI references policy-bound security); §10 compliance guarantees ↔ §11 audit requirements
- **External dependencies**: §10 data principles overlap with Cluster C

#### Cluster C: Architecture, Data & Integration (§8, §9, §10, §13, §14, §15)
- **Content**: Platform/tooling, architectural approach, data principles, integration philosophy, scalability, technical non-goals
- **Words**: ~1,340 | **Tokens**: ~1,780
- **Consumers**: Architect, Database Administrator, Spec Writer (§15)
- **Update frequency**: Medium (evolves as platform matures through stages)
- **Internal cross-references**: High (§9 references §13 integration; §14 references §9 modular monolith; §8 informs §9)
- **External dependencies**: §10 data principles shared with Cluster B

#### Cluster D: Change Policy (§16)
- **Content**: What's stable vs. evolving, review cadence, revision triggers
- **Words**: ~210 | **Tokens**: ~280
- **Consumers**: All agents that read any vision content (meta-governance)
- **Update frequency**: Very low
- **External dependencies**: Governs all other sections — must be appended to or referenced from every split document

### 2.2 Cross-Reference Density Map

```
§1-§7 (Product)     §8-§9,§14-§15 (Architecture)     §10 (Data)     §11 (Security)     §12 (AI)     §13 (Integration)
      │                        │                          │                │                 │                │
      └── §7 non-goals ──────►│ §15 tech non-goals       │                │                 │                │
                               │◄─────────────────────────┤                │                 │                │
                               │  §9 references §10       │◄───────────────┤                 │                │
                               │  data flow rules         │  §10 compliance│                 │                │
                               │                          │  guarantees ──►│ §11 audit reqs  │                │
                               │                          │                │◄────────────────┤                │
                               │                          │                │  §12 references  │                │
                               │                          │                │  policy-bound ──►│                │
                               │◄─────────────────────────┼────────────────┼─────────────────┼────────────────┤
                               │  §9 references §13       │                │  §11 references  │  §13 policy-  │
                               │  integration layer       │                │  trust boundaries│  gated actions │
```

**Key finding**: §10 (Data Principles) is the highest-density cross-reference node. It connects to:
- §11 (compliance guarantees → audit requirements)
- §9 (data flow rules → architectural approach)
- §13 (sync model → integration philosophy)
- §12 (PHI handling → AI boundaries)

**Implication**: §10 cannot cleanly live in only one split document. The compliance-relevant portions (lines 211–216: "Compliance & Auditability Guarantees") should be included in the security/compliance document, while the full section belongs in the architecture document. This is the **one justified duplication** in a decomposition strategy.

---

## 3. Agent Impact Assessment

### 3.1 Current Cost: Loading Full Document

If the governance report's recommendations are implemented (7 agents read vision), the token cost per orchestration cycle would be:

| Scenario | Agents Loading | Tokens Per Agent | Total Tokens |
|----------|---------------|-----------------|--------------|
| Full document, 7 agents | 7 | 4,400 | 30,800 |
| Full document, 12 agents (future) | 12 | 4,400 | 52,800 |

### 3.2 Projected Cost: Domain-Split Documents

| Document | Tokens | Agents Loading | Total Tokens |
|----------|--------|---------------|--------------|
| `vision-security-compliance.md` (§10 compliance, §11, §12) | ~1,100 | 4 (security + compliance quartet) | 4,400 |
| `vision-architecture.md` (§8, §9, §10, §13, §14, §15) | ~1,780 | 2 (architect, DBA) | 3,560 |
| `vision-product-strategy.md` (§1–§7) | ~1,260 | 1 (spec-writer, §7 only) | 1,260 |
| **Total** | | **7 agents** | **9,220** |

**Token savings: 30,800 → 9,220 = 70% reduction** (21,580 tokens saved per orchestration cycle)

### 3.3 Impact on Agent Read-Contracts

| Agent | Current Reads | Proposed Addition | Token Cost |
|-------|--------------|-------------------|------------|
| Compliance Engineer | specs, tasks, skills | + `vision-security-compliance.md` | +1,100 |
| Compliance Auditor | PR diff, compliance.md, specs, tasks, skills | + `vision-security-compliance.md` | +1,100 |
| Security Engineer | specs, tasks, skills | + `vision-security-compliance.md` | +1,100 |
| Security Auditor | PR diff, security.md, config, deps, skills | + `vision-security-compliance.md` | +1,100 |
| Architect | prd, specs (skim), system-design, skills | + `vision-architecture.md` | +1,780 |
| Database Administrator | specs, tasks, schema, skills | + `vision-architecture.md` (§10, §14 only) | +665* |
| Spec Writer | prd, spec-change-requests, system-design, skills | + `vision-product-strategy.md` (§7) + `vision-security-compliance.md` (§11, §12) | +1,000* |

*DBA and Spec Writer could use section-specific reads rather than full split documents for additional savings.

---

## 4. Decomposition Recommendation

### 4.1 Recommended Structure: Domain-Specific Split (4 documents)

**Approach**: Split into 4 focused documents under `.ops/` plus keep the original as the canonical source.

#### Document 1: `.ops/vision-product-strategy.md`
**Contains**: §1 Vision, §2 Problem Space, §3 Target Customers, §4 Value Proposition, §5 Strategic Pillars, §6 Success Metrics, §7 Non-Goals, §16 Change Policy (appended)
**Size**: ~1,540 tokens
**Primary consumers**: Spec Writer (non-goals), UI Designer (pillars), product stakeholders
**Standalone coherence**: High — fully self-contained product strategy

#### Document 2: `.ops/vision-security-compliance.md`
**Contains**: §11 Security & Compliance Posture, §12 AI Strategy, §10 Compliance & Auditability Guarantees (lines 211–216 only), §16 Change Policy (appended as reference)
**Size**: ~1,200 tokens
**Primary consumers**: Security Engineer, Security Auditor, Compliance Engineer, Compliance Auditor
**Standalone coherence**: High — all security/compliance/AI rules in one place

#### Document 3: `.ops/vision-architecture.md`
**Contains**: §8 Core Platform & Tooling, §9 Architectural Approach, §10 Data Principles (full), §13 Integration Philosophy, §14 Scalability Intent, §15 Technical Non-Goals, §16 Change Policy (appended as reference)
**Size**: ~2,060 tokens
**Primary consumers**: Architect, Database Administrator
**Standalone coherence**: High — complete technical strategy

#### Document 4: `.ops/product-vision-strategy.md` (original, unchanged)
**Role**: Canonical source of truth. Split documents are derived views.
**When to read**: Full document only needed for cross-domain analysis, quarterly reviews, or onboarding
**Size**: ~4,400 tokens (unchanged)

### 4.2 Section Allocation Map (All 16 Sections Accounted For)

| § | Section | Primary Document | Also Referenced In |
|---|---------|------------------|--------------------|
| 1 | Vision | `vision-product-strategy.md` | — |
| 2 | Problem Space | `vision-product-strategy.md` | — |
| 3 | Target Customers | `vision-product-strategy.md` | — |
| 4 | Value Proposition | `vision-product-strategy.md` | — |
| 5 | Strategic Pillars | `vision-product-strategy.md` | — |
| 6 | Success Metrics | `vision-product-strategy.md` | — |
| 7 | Non-Goals | `vision-product-strategy.md` | — |
| 8 | Core Platform & Tooling | `vision-architecture.md` | — |
| 9 | Architectural Approach | `vision-architecture.md` | — |
| 10 | Data Principles | `vision-architecture.md` (full) | `vision-security-compliance.md` (§10.4 only*) |
| 11 | Security & Compliance Posture | `vision-security-compliance.md` | — |
| 12 | AI Strategy | `vision-security-compliance.md` | — |
| 13 | Integration Philosophy | `vision-architecture.md` | — |
| 14 | Scalability Intent | `vision-architecture.md` | — |
| 15 | Technical Non-Goals | `vision-architecture.md` | — |
| 16 | Change Policy | `vision-product-strategy.md` (full) | `vision-security-compliance.md` (ref), `vision-architecture.md` (ref) |

*§10.4 = "Compliance & Auditability Guarantees" subsection (lines 211–216) — the only justified duplication, since compliance agents need these guarantees but don't need full data flow rules.

### 4.3 Cross-Reference Handling

Each split document includes a header:

```markdown
<!-- Derived from: .ops/product-vision-strategy.md -->
<!-- Sections: §X, §Y, §Z -->
<!-- Canonical source: .ops/product-vision-strategy.md -->
<!-- Related: .ops/vision-{other}.md for [domain] context -->
```

This provides:
- **Traceability**: Clear lineage to the canonical source
- **Discoverability**: Agents know where to find related context if needed
- **No circular dependencies**: Split documents reference the parent, not each other

---

## 5. Trade-offs and Risks

### 5.1 Benefits of Decomposition

| Benefit | Impact |
|---------|--------|
| 70% token reduction per orchestration cycle | ~21,580 tokens saved across 7 agents |
| Domain-scoped loading | Agents load only relevant context |
| Closes governance gap | Split documents can be added to agent read-contracts directly |
| Reduced cognitive overhead | New contributors consult domain-specific docs |
| Independent update cadence | Security posture changes don't require product strategy review |

### 5.2 Risks of Decomposition

| Risk | Severity | Mitigation |
|------|----------|------------|
| **Drift between split docs and canonical source** | Medium | Treat split docs as derived artifacts; regenerate from canonical source. Add a note in Change Policy: "Update canonical source first, then regenerate split documents." |
| **§10 duplication** (compliance guarantees in two docs) | Low | Only 6 lines (~80 tokens) duplicated. Clearly marked with "Source: §10.4" in both locations. |
| **New contributors not knowing which doc to consult** | Low | CLAUDE.md pointer (see recommendation) directs agents; canonical source available for full context. |
| **File proliferation under `.ops/`** | Low | Only 3 new files (net). Clear naming convention: `vision-{domain}.md`. |
| **Change Policy must govern all split docs** | Low | §16 appended to or referenced in every split document. |

### 5.3 Comparison of Approaches

| Approach | Token Savings | Migration Effort | Drift Risk | Agent Simplicity |
|----------|--------------|-----------------|------------|-----------------|
| **Keep monolithic** (status quo) | 0% | None | None | Low (one file to reference) |
| **Minimal split** (2 docs: product + technical) | ~35% | Low | Low | Medium |
| **Domain-specific split** (4 docs) — **Recommended** | ~70% | Medium | Medium | High (agents get exactly what they need) |
| **Section-tagged monolith** (machine-readable markers) | ~60%* | Low | None | Medium (requires parsing logic) |

*Section-tagged approach depends on agents being able to extract sections programmatically, which isn't natively supported.

---

## 6. Migration Impact

### 6.1 Agent File Updates Required

These changes align with the governance report's P1–P3 recommendations:

| Agent File | Change | Priority |
|-----------|--------|----------|
| `claude/agents/compliance-engineer.md` | Add `vision-security-compliance.md` to Reads | P1 |
| `claude/agents/compliance-auditor.md` | Add `vision-security-compliance.md` to Reads | P1 |
| `claude/agents/security-engineer.md` | Add `vision-security-compliance.md` to Reads | P1 |
| `claude/agents/security-auditor.md` | Add `vision-security-compliance.md` to Reads | P1 |
| `claude/agents/spec-writer.md` | Add `vision-product-strategy.md` (§7) + `vision-security-compliance.md` to Reads | P2 |
| `claude/agents/architect.md` | Add `vision-architecture.md` to Reads | P2 |
| `claude/agents/database-administrator.md` | Add `vision-architecture.md` (§10, §14) to Reads | P3 |

### 6.2 CLAUDE.md Update

**Current** (line 83):
```
- Do NOT read `.ops/product-vision-strategy.md` or `.ops/ui-design-system.md` unless needed
```

**Recommended replacement**:
```
- `.ops/product-vision-strategy.md` — Full vision document. Do NOT load unless doing cross-domain analysis or quarterly review.
  - Domain splits (preferred for agents):
    - `.ops/vision-product-strategy.md` — Product vision, pillars, non-goals (§1–§7, §16)
    - `.ops/vision-security-compliance.md` — Security posture, compliance, AI boundaries (§10.4, §11, §12, §16 ref)
    - `.ops/vision-architecture.md` — Architecture, data, integration, scalability (§8–§10, §13–§15, §16 ref)
  - Agents should load only their relevant domain split, not the full document
- Do NOT read `.ops/ui-design-system.md` unless doing UI work
```

### 6.3 New File Creation

| File | Source Sections | Action |
|------|----------------|--------|
| `.ops/vision-product-strategy.md` | §1–§7, §16 | Create (derived from canonical source) |
| `.ops/vision-security-compliance.md` | §10.4, §11, §12, §16 ref | Create (derived from canonical source) |
| `.ops/vision-architecture.md` | §8–§10, §13–§15, §16 ref | Create (derived from canonical source) |

---

## 7. Final Recommendation

**Decompose using the domain-specific split (4-document approach).**

### Rationale

1. **Closes the governance gap** — The governance report identified 7 agents that critically need vision content but load none. Split documents make it practical to add domain-scoped reads without the 4,400-token cost of the full document.

2. **70% token reduction** — Across a 7-agent orchestration cycle, this saves ~21,580 tokens. As the agent count grows, savings compound.

3. **Natural domain boundaries** — The 16 sections cluster cleanly into product strategy, security/compliance, and architecture domains with minimal cross-reference overhead (only §10.4 requires duplication).

4. **Low drift risk** — The canonical source remains unchanged. Split documents are clearly marked as derived artifacts with regeneration instructions.

5. **Incremental adoption** — Can implement in phases: P1 agents (security/compliance quartet) first with `vision-security-compliance.md`, then P2 (architect, spec-writer), then P3 (DBA).

### Implementation Order

1. Create the 3 split documents from the canonical source
2. Update P1 agent files (compliance-engineer, compliance-auditor, security-engineer, security-auditor)
3. Update CLAUDE.md pointer
4. Update P2 agent files (architect, spec-writer)
5. Update P3 agent files (database-administrator)

---

## Appendix: Validation Checklist

- [x] Every section of the original doc is accounted for in the proposed structure (see §4.2 allocation map)
- [x] No information is lost across split documents (canonical source preserved)
- [x] Only one justified duplication: §10.4 (~80 tokens, 6 lines) in both architecture and security/compliance docs
- [x] Cross-references between split docs are explicitly documented (see §4.3)
- [x] Agent read-contracts are updated in the recommendation (see §6.1)
- [x] Token savings are quantified: ~70% reduction, ~21,580 tokens saved per 7-agent cycle
- [x] The change-policy section (§16) is preserved: full copy in product-strategy doc, reference in other two
- [x] CLAUDE.md token-conservation rules are updated in recommendations (see §6.2)

---

*Generated with Claude Code Specialist analysis*
*Cross-referenced with: agent-governance-report.md (2026-02-05)*
