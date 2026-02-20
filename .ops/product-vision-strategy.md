# Product Vision & Strategy

## 1. Vision

Enable small therapy practices to run their entire business from Google Workspace, eliminating the need for separate practice management tools.

**Long-term platform vision:** Enable regulated teams (starting with therapy practices) to use AI and modern tools within Google Workspace without risking compliance, trust, or future rework.

**Product evolution strategy:**
- **Practice Management (Wedge)** → Solve immediate, daily problems (notes, scheduling, billing, communication); prove trust with sensitive data; establish Curio as part of core operations
- **Policy Management (Control Layer)** → Encode compliance rules as system policies; govern access, data flow, retention, and AI behavior
- **Compliance Infrastructure (Platform)** → Abstract policies into reusable primitives; provide auditability, enforcement, and verification at scale; become the trusted compliance layer for AI-enabled work across regulated industries

---

## 2. Problem Space

**Primary pain points:**
1. Solo online therapists spend too much time on admin tasks (therapy notes, processing payments, sending invoices, insurance billing, communicating with clients, managing their agenda)
2. Existing practice management tools are expensive for solo practitioners
3. Existing tools are complex and too generic, not covering online therapy use cases (group therapy, insurance billing, workflows)
4. Therapists are familiar with Google Workspace and spend most of their time in it (Gmail, Calendar, Google Docs)

**Who experiences these problems:** Solo online therapists (one therapist, fully remote practice) and solo therapists (one therapist, hybrid practice)

**Why existing solutions fall short:**
- Too expensive for solo practitioners
- Too complex/feature-heavy
- Too generic, not built for online therapy workflows
- Require switching away from tools therapists already use (Google Workspace)

---

## 3. Target Customers

**Primary target:** Solo online therapists (one therapist, fully remote practice)

**Secondary target:** Solo therapists (one therapist, hybrid practice)

**Focus:** Primary and secondary for now

**Future target:** Small practices (2-5 therapists, hybrid practice) - later

**Key characteristics:**
- Familiar with Google Workspace (Gmail, Calendar, Google Docs)
- Spend most of their time in Google Workspace
- Need online therapy-specific features (group therapy, insurance billing, workflows)
- Cost-sensitive (solo practitioners)

---

## 4. Value Proposition

**Core Benefits:**
- **Everything in one place** - Sessions, notes, calendar, documents, and client communication live together — no tool juggling, no copy-pasting, no missed details
- **Compliance without the stress** - Curio is designed to meet healthcare privacy requirements by default, so you don't have to figure out settings, policies, or legal details on your own
- **AI that actually saves time** - Notes, summaries, and admin support are automated in a way that's safe for client data — helping you reclaim hours each week
- **Works with tools you already use** - Curio fits naturally into Google Workspace, so you don't need to learn an entirely new system or migrate everything
- **Built for online therapy** - Designed from day one for telehealth, not retrofitted from in-person workflows

**Why Not the Alternatives?**

Most alternatives either:
- Focus on features but leave compliance up to you
- Add AI without clear data protections
- Require multiple disconnected tools
- Feel heavy, outdated, or built for clinics—not solo therapists

**Differentiation:** Curio is different because it's simple on the surface and safe underneath.

**One-Line Value Proposition:** Curio helps solo online therapists run their practice and save time — without worrying about compliance, tech complexity, or client data safety.

**Platform vision (long-term):**
- **Compliance by default** - Safety and regulatory guarantees are built into the system, not left to settings or training
- **AI you can actually trust** - AI operates within clear, auditable boundaries, making it safe for real, regulated work
- **No new silos** - Curio governs work inside tools you already use (like Google Workspace), reducing risk and complexity
- **Built to scale safely** - Growth doesn't increase compliance burden; audits and governance get easier over time

**Intentional Trade-Offs:**
- Less flexibility, more certainty - You can't override safety-critical rules
- Fewer flashy features - Only AI and automations that are safe in production
- Opinionated by design - Strong defaults so you don't need to be a compliance expert

---

## 5. Strategic Pillars

1. **Fits Into Therapists' Daily Work** - Curio adapts to existing workflows and tools (Google Workspace and others) instead of forcing therapists into a new silo

2. **Compliance by Default** - Regulatory and privacy requirements are enforced by the system, not left to configuration or user behavior

3. **AI with Clear Boundaries** - AI is policy-bound, auditable, and safe for real clinical use

4. **Radical Simplicity for Solo Therapists** - Every feature must reduce admin time and cognitive load

5. **Built to Scale Beyond Practice Management** - Product decisions must support Curio's evolution into a broader compliance platform

---

## 6. Success Metrics

**The Single Proof It's Working:** Therapists stop thinking about compliance and admin — and simply run their practice through Curio every week.

**North Star Metric:** Weekly Active Therapists running core workflows end-to-end in Curio (e.g. sessions → notes → follow-ups → billing, without leaving their existing tools)

If Curio is working, therapists default to it to run their practice.

**Leading Indicators (Wedge: Therapy Practices):**
- Net admin time saved per therapist per week → Direct signal of real value delivered
- 30/90-day retention of solo therapists → Indicates Curio is embedded in daily work
- AI feature adoption within compliant workflows → Signals trust, not just curiosity
- Self-reported compliance confidence ("I feel safe using Curio with client data") → Measures peace of mind, not feature usage
- Revenue per active therapist → Confirms willingness to pay for value, not just access

**Long-Term Platform Signals:**
- % of workflows governed by explicit policies → Measures transition from features to compliance primitives
- Number of reusable policy types supported → Indicates platform maturity
- Expansion into adjacent regulated roles (without rebuilding core systems) → Proof of horizontal scalability

---

## 7. Non-Goals

- Not for large clinics/hospitals
- Not for non-regulated industries
- Not a full EHR
- Not a billing-only tool
- Not a general productivity tool
- Not supporting non-Google Workspace platforms (initially)

---

# Technical Strategy (High Level)

## 8. Core Platform & Tooling

**Frontend:** Next.js, TypeScript, Tailwind CSS, shadcn, headless UI (Tailwind Labs)

**Backend/Database:** Supabase

**Deployment:** Vercel

**Payments:** Stripe

**Email:** Resend

**Analytics/Testing/Feature Flags:** PostHog

**Error Monitoring:** Sentry

**Google Workspace Integration Evolution:**
1. **Embedded Workflow Integration (In-Context)** - Curio actions inside Calendar, Gmail, Docs; generate notes, summaries, follow-ups where therapists already work; Curio adapts to existing workflows
2. **Governance & Policy Layer (Control Plane)** - Curio manages permissions, data access, retention, and AI usage across Workspace; policies applied across tools, not per feature; audit logs and enforcement at the system level
3. **Compliance-Native Workspace Mode (Long-Term)** - Curio becomes the compliance abstraction for Workspace; therapists work normally; Curio enforces guarantees invisibly; AI, access, and data flows are policy-bound by default

**Evolution:** Start at #1 → evolve into #2 → grow into #3. Meet therapists where they already work first. Add governance next. Become infrastructure over time.

---

## 9. Architectural Approach

**Guiding Philosophy:** Build a compliance-first core that can start as a product (practice management) and naturally harden into a platform (policy + governance) without rewrites.

**High-Level Choices:**

1. **Modular Monolith first (with hard module boundaries)** - Start as a modular monolith to ship fast and keep data/workflows consistent. Enforce boundaries like microservices internally (separate domains, clear interfaces). Extract services later only when scale/security demands it.

2. **API-first and Integration-first** - Treat every capability as an API (even if you have a UI). Build an "integration layer" as a first-class subsystem (Google Workspace + others), not ad-hoc connectors.

3. **Policy Engine as the Core Abstraction** - Centralize decisions in a policy engine (permissions, data flow, retention, AI boundaries). All actions (human or AI) go through the policy layer.

4. **Event-driven for auditability and extensibility** - Use an event log as the system of record for "who did what, when, with what data, under what policy." Workflows become orchestrations over events (reliable, replayable, observable).

5. **Hybrid compute (simple runtime, scalable edges)** - Keep the core simple and reliable (traditional app runtime is fine). Use serverless/async jobs for bursty workloads (AI processing, webhooks, background sync).

**How It Evolves Across the 3 Stages:**

- **Stage 1: Practice Management (Wedge)** - One core domain model for therapist workflows. Integration layer connects Workspace (calendar, docs, email) to workflows. Event log + audit exists from day one (even if lightweight).

- **Stage 2: Policy Management (Control Layer)** - Promote policy engine from "permissions" to governance: AI allowed/blocked behaviors, data access + sharing rules, retention and disclosure controls. Policies become configurable primitives, not feature-specific logic.

- **Stage 3: Compliance Infrastructure (Platform)** - Policy primitives become reusable across industries. Externalize capabilities: Policy APIs, audit/attestation exports, integration SDK / connector framework. Selectively split components (policy, audit, integrations) into services only when needed.

**The Core Rule (Architectural North Star):** Every action is policy-checked and audit-recorded. That's what lets Curio grow from "app that helps" to "infrastructure you trust."

---

## 10. Data Principles

**Ownership and Source of Truth (SoR):**

- **Identity / auth / groups:** Google Workspace is SoR (Curio mirrors minimal identifiers/roles)
- **Scheduling (calendar events):** Google Calendar is SoR; Curio stores references + derived metadata (never forks events)
- **Clinical notes / summaries / AI outputs:** Curio is SoR (because governance, retention, and audit must be guaranteed). Curio may export a copy to Drive/Docs only when explicitly enabled
- **Client record (PHI profile, consents, preferences):** Curio is SoR
- **Messages (email):** Gmail is SoR; Curio stores thread IDs + permitted excerpts/metadata and logs actions taken
- **Files (documents):** Drive is SoR for the binary; Curio stores file IDs, hashes, permissions snapshot, and policy labels

**Data Flow Rules:**

- **Workspace → Curio:** Ingest via webhooks/polling into an immutable event log; materialize read models for UX
- **Curio → Workspace:** Write only through policy-checked "export/publish" actions (e.g., create calendar event, send email, create doc)
- **No silent writes:** Curio never modifies Workspace data without an explicit user/system action + audit entry

**Sync vs Duplication:**

- **Acceptable duplication (with strict controls):** IDs, timestamps, minimal metadata, permission snapshots; derived data (indexes, summaries, embeddings) that can be regenerated; encrypted cached content for performance only with TTL + revalidation
- **Must stay single-source (no forks):** Calendar events, Drive files, Gmail threads (Curio references them, doesn't duplicate as primary)
- **Allowed "dual-home" with explicit choice:** Clinical notes can be stored in Curio and optionally exported to Drive as a controlled copy (treated as an external artifact, not SoR)

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

## 13. Integration Philosophy

**Core Approach:**

- **Native-first for the wedge:** Deep, reliable Google Workspace integrations (Calendar/Gmail/Drive/Meet) to fit therapist workflows with minimal change
- **Connector framework for scale:** Evolve to a generic integration layer (providers, webhooks, mappings) once the policy engine is mature

**Sync Model:**

- **Prefer references over copies:** Keep external systems as SoR where they already are (e.g., Calendar/Gmail/Drive), store IDs + minimal metadata + audit/provenance
- **Two-way only when necessary:** Start one-way ingest + explicit publish (Curio writes only via user-approved actions). Move to selective two-way sync when conflict handling + audit are solid
- **Idempotent + replayable:** Integrations must be resilient (dedupe, retries, backoff) and reconstructable from an event log

**Trust Boundaries (Non-Negotiable):**

- **External systems are untrusted inputs:** Validate, normalize, and policy-check everything coming in
- **Never delegate compliance to third parties:** Curio enforces policy before any outbound write/export
- **Least-privilege scopes:** Minimal OAuth scopes per connector; per-tenant isolation; short-lived tokens; secret rotation
- **Data minimization:** Do not pull/store PHI unless required; encrypt any cached content; TTL where possible

**Governance Model:**

- **Policy-gated integration actions:** Every read/write is authorized by the policy engine (actor, purpose, data class, destination)
- **Auditable by default:** Log all integration events (who/what/when/where/why, payload hashes, outcome)
- **Connector certification:** Define tiers (trusted/verified/custom) with required security controls and ongoing monitoring

**How It Supports the 3 Stages:**

- **Practice management:** Workspace-native, low-friction, minimal data duplication
- **Control layer:** Policy engine governs all connector behavior (exports, sharing, retention)
- **Platform:** Standardized connector SDK + reusable policy primitives to onboard new regulated verticals and systems (EHRs, payments, e-sign, messaging) without redesign

---

## 14. Scalability Intent

**Scalability Philosophy:** Scale correctness and trust before throughput. In a compliance product, auditability, isolation, and determinism matter more than raw QPS. Design for scale via boundaries, not complexity. Start simple (modular monolith + queues), but enforce clear domain/module interfaces so you can split later without rewrites.

**Early-Stage Priorities (Solo Therapists):**

- **Simplicity > micro-optimizations:** Optimize developer velocity and reliability
- **Async-first for heavy work:** AI processing, sync, indexing, and exports run in background jobs; UI stays fast
- **Tenant isolation + rate limits from day one:** Prevent noisy-neighbor issues early

**When to Optimize vs Keep Simple:**

Optimize only when you hit one of these triggers:
- Customer-visible latency (core workflows feel slow)
- Reliability risk (timeouts, backlog growth, missed webhooks)
- Cost blowups (AI or sync costs scale non-linearly)
- Compliance risk (logs incomplete, retention/audit failing)
- Operational load (on-call pain, frequent incidents)

If none apply, keep it simple.

**How Curio Scales from Wedge → Platform:**

- **Vertical scaling:** Strengthen caching, indexing, and job queues; keep monolith
- **Horizontal scaling:** Split by workload type, not by "service ideology": ingestion/sync workers, AI pipeline workers, policy/audit subsystem
- **Platform scaling:** Extract only the stable, shared primitives into services: policy engine, audit/event ledger, integration framework, identity/entitlements
- **Multi-tenant hardening:** Per-tenant keys, regional pinning, quotas, and isolation become first-class

**Rule of Thumb:** Ship fast with a modular core; scale by separating hot paths and risky domains (sync, AI, policy, audit) into independently scalable components as demand proves it.

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

---

*Generated with Clavix Product Strategy Mode*
*Generated: 2025-01-27*
*This document informs all PRDs and should remain stable across releases*
