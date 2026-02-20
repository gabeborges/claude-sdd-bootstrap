<!-- Source: .ops/product-vision-strategy.md -->
<!-- Included: §10 (partial: Data Flow Rules + Compliance & Auditability Guarantees), §11, §12, §15, §16 -->
<!-- Skipped: none -->
<!-- Regenerate: /vision:distill -->

## 10. Data Principles (Compliance-Relevant Subsections)

**Data Flow Rules:**

- **Workspace → Curio:** Ingest via webhooks/polling into an immutable event log; materialize read models for UX
- **Curio → Workspace:** Write only through policy-checked "export/publish" actions (e.g., create calendar event, send email, create doc)
- **No silent writes:** Curio never modifies Workspace data without an explicit user/system action + audit entry

**Compliance & Auditability Guarantees:**

- Every read/write across boundaries is policy-gated, least-privilege, and logged (actor, purpose, data scope, outcome)
- Curio maintains provenance: which Workspace objects influenced a note/AI output and under which policy
- Prefer minimize PHI duplication; when duplicated, it must be encrypted, scoped, time-bounded, and revocable where possible

---

## 11. Security & Compliance Posture

**Security Assumptions That Must Always Hold:**

- **Zero trust by default:** Every request is authenticated, authorized, and policy-checked; no implicit trust by network or environment
- **Least privilege everywhere:** Scoped OAuth for Workspace; tight RBAC/ABAC in Curio; short-lived tokens; just-in-time access for ops
- **All actions are attributable:** Every read/write/AI action has an actor (human/service), purpose, policy decision, and trace ID
- **Secure-by-default configs:** Customers cannot misconfigure Curio into an unsafe/non-compliant state

**Access Control Philosophy:**

- Workspace identity is authoritative (SSO via Google); Curio maps roles/entitlements with explicit scoping
- Break-glass is controlled (time-bound, approval-gated, fully logged, alerting enabled)
- Separation of duties for admin/security/billing roles; no "god admin" by default

**Audit Requirements:**

- Immutable audit log for security-relevant events (auth, access, exports, permission changes, AI processing, data lifecycle)
- Tamper-evident storage + retention policies; exportable for audits (SOC2 evidence, incident response, customer access logs)
- Provenance tracking for AI outputs (inputs, model/provider, policy context, data sources)

**Encryption Requirements:**

- Encryption in transit (TLS 1.2+; modern ciphers)
- Encryption at rest for all Curio-managed data (KMS-backed, key rotation)
- Field-level encryption for PHI/PII and secrets; separate keys per tenant where feasible

**Data Residency / Sovereignty:**

- Region pinning per tenant (e.g., Canada for PHIPA, EU for GDPR where required)
- No cross-region replication of PHI unless explicitly contracted + compliant
- Subprocessor controls with DPAs/BAAs and clear data flow boundaries

**Non-Negotiable Compliance Commitments:**

- **HIPAA / PHIPA readiness:** BAAs where applicable, administrative/technical safeguards, breach response procedures
- **SOC 2 Type II as baseline:** Security + Availability; add Confidentiality/Privacy as needed
- **GDPR where applicable:** DPIAs, lawful basis, DSR workflow, minimization, purpose limitation

**Evolution: Wedge → Platform:**

- **Wedge:** Ship with the full posture above, even if scope is small (audit, least privilege, encryption, residency)
- **Platform:** Add policy engine enforcement, automated evidence collection, continuous controls monitoring, and standardized attestations; expand residency options and tenant-isolated cryptography

---

## 12. AI Strategy

**Role of AI:** AI is an assistive copilot, not an autonomous clinician. It accelerates admin work (notes, summaries, follow-ups, billing drafts) while staying policy-bound and auditable.

**Non-Negotiable Boundaries & Guarantees:**

- **Policy-bound execution:** AI can only read/write what the policy engine allows for that actor/context
- **Explainability + provenance:** Every output links to inputs/sources + the policy decision that permitted it
- **No silent actions:** AI never sends, files, bills, shares, or deletes without explicit user confirmation (or an admin-approved, policy-defined automation)
- **No training on customer PHI:** Customer data is not used to train shared models
- **Fail-closed:** If policy, identity, or context is unclear, AI refuses or asks

**Must Always Have Human Oversight:**

- Clinical notes finalization (sign-off required)
- Any external communication (emails/messages to clients)
- Billing/claims submission and financial actions
- Sharing/exporting PHI outside Curio (Drive, email, third parties)
- Permission/policy changes and retention/deletion actions

**What AI Explicitly Cannot Do:**

- Diagnose, prescribe, or provide clinical advice as a professional replacement
- Change policies/permissions, bypass access controls, or expand its own scope
- Access data across clients/tenants beyond authorized context
- Create "new workflows" that aren't defined in product + policy

**Fit with Compliance-First Architecture:**

- AI is treated as a constrained service behind the policy engine
- All AI operations emit audit events (inputs, outputs, model/provider, purpose, actor, policy decision)
- Outputs are draft artifacts until user approval (human-in-the-loop as a default safety rail)

**Evolution Across the 3 Stages:**

- **Practice management:** Assistive drafting + summaries + task suggestions; heavy human review
- **Policy management:** AI becomes "policy-aware" (suggests next steps that comply; flags risky actions)
- **Compliance platform:** AI helps generate evidence, audits, and policy checks (e.g., "show all exports last 30 days"), still policy-bound and non-autonomous for high-risk actions

---

## 15. Technical Non-Goals

- No on-prem / self-hosting in the early stages (cloud/SaaS only, controlled environment for compliance)
- No non–Google Workspace-first platform initially (Microsoft 365 and others come later)
- No multi-workspace abstraction early (avoid building a generic "works with everything" layer before the policy engine is mature)
- No microservices-first architecture (start modular monolith + async workers; extract later only with clear scale/security triggers)
- No offline-first or local PHI storage (mobile/desktop offline caches of sensitive data are out)
- No real-time collaborative editing (Google Docs remains the collaboration surface; Curio doesn't replicate it)
- No "AI autopilot" (no unsupervised sending, billing, sharing, deletion, or policy changes)
- No broad connector marketplace early (only vetted, first-party integrations; partner/SDK later)
- No legacy browser support beyond modern evergreen browsers (avoid IE/old Safari compatibility work)
- No storing full duplicates of Workspace content by default (reference/metadata-first; export copies only when explicitly enabled)

---

## 16. Change Policy

**Must Remain Stable (Non-Negotiables):**

- Compliance-first posture (safety, auditability, least-privilege, fail-closed)
- Policy engine as the enforcement layer for humans + AI
- Workspace-native philosophy (fit into therapist workflows; avoid silos)
- Human oversight for high-risk actions (send/share/bill/delete/policy changes)
- Data minimization + clear source-of-truth boundaries

**May Evolve (Expected to Change):**

- Wedge scope & packaging (features, pricing, target sub-segment within therapists)
- Technical shape (modular monolith → selective services; compute choices)
- AI capability depth (models/providers, evaluation methods, UI patterns)
- Integration breadth (beyond Google Workspace; connector framework maturity)
- Compliance targets (SOC2 scope expansion, regional requirements, certifications)

**Review Cadence:**

- **Quarterly:** Validate strategy + pillars + assumptions (market, wedge fit, traction)
- **Annually:** Re-baseline architecture/compliance roadmap (SOC2, residency, platform expansion)
- **Immediate review after any major trigger below**

**Triggers for Major Revision:**

- Regulatory shift impacting data handling (PHIPA/HIPAA/GDPR changes, new guidance)
- Security incident or near-miss that challenges assumptions
- Platform change by Google (API, OAuth scopes, Workspace policy changes)
- Wedge failure (weak retention/time-saved, low willingness-to-pay)
- Expansion readiness (consistent PMF + demand from adjacent regulated verticals)
- Material AI risk (unacceptable hallucination/traceability gaps, new model constraints)

**Rule:** Pillars/invariants change rarely; execution details change often.
