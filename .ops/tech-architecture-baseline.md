<!-- Source: .ops/product-vision-strategy.md -->
<!-- Included: §8, §9, §10, §12, §13, §14, §15, §16 -->
<!-- Skipped: none -->
<!-- Regenerate: /vision:distill -->

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
