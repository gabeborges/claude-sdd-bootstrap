# Decisions Log

> Append-only. Each entry records a decision, state change, or deviation made during the build.
> Entries are never deleted, modified, or reordered.

---

## [2026-02-06 00:00] Orchestrator — v0 Build Orchestration Initiated

**Changed**: Build orchestration started for v0 (Booking & Scheduling)
**Decision**: PRD review complete. v0 scope targets a booking and scheduling system for solo online therapists (Canada only). The PRD defines 7 must-have features: service/appointment CRUD, availability management, public booking page, booking management, booking emails and reminders, Google Calendar sync, and online/in-person support. Key constraints include HIPAA/PHIPA compliance from day 0, Canada-only data residency, and no silent writes to Google Workspace. Stack: Next.js (App Router), Supabase, Google OAuth, Google Calendar API, Resend for email. PRD-detected agent domains: UI/frontend, security (HIPAA/PHIPA, encryption, audit logging), compliance (data residency, PHI handling), database (Supabase schema, RLS, tenant isolation).
**Follow-ups**: Tier 2 agents (spec-writer, architect, database-administrator, project-task-planner) to produce specs.md, system-design.yaml, db-migration-plan.yaml, and tasks.yaml in sequence.

---

## [2026-02-06 00:00] Orchestrator — Agent Tier Plan Established

**Changed**: Orchestration tier assignments defined for v0 build
**Decision**: Based on PRD keyword analysis, the following tier plan was established:
- **Tier 2 (Sequential — Design Phase):** spec-writer -> architect -> database-administrator -> project-task-planner. Sequential because each depends on the prior artifact (specs.md -> system-design.yaml -> db-migration-plan.yaml -> tasks.yaml).
- **Tier 3 (Parallel — Specialist Review):** ui-designer, security-engineer, compliance-engineer. These run in parallel once Tier 2 artifacts are complete.
- **Tier 4 (Frontend Design):** frontend-designer. Depends on ui-designer output from Tier 3.
- **Tier 5 (Implementation):** fullstack-developer, test-automator. Build and test against Tier 2-4 artifacts.
- **Tier 6 (Audit & QA):** qa, code-reviewer, security-auditor, compliance-auditor. Final validation gate.

Rationale: Security and compliance agents detected from PRD constraints (HIPAA/PHIPA, data residency, audit logging, encryption). UI agent detected from public booking page and therapist setup flows. Database agent detected from Supabase usage, RLS requirements, and tenant isolation patterns.
**Follow-ups**: Tier 2 begins with spec-writer. No artifacts exist yet beyond the PRD.

---

## [2026-02-06 13:30] Spec Writer — 7 Feature Specs Produced for v0

**Changed**: Created specs.md for all 7 features: booking-services, availability, booking-page, booking-management, booking-notifications, google-calendar-sync, therapist-setup
**Decision**: 115 requirements and 75 acceptance scenarios cover the full v0 scope. 24 open questions flagged as design-gate items (not blockers). Cross-cutting concerns — HIPAA/PHIPA, Canada data residency, no silent writes, Supabase RLS, WCAG, AI non-goals — are encoded directly into feature specs rather than a separate cross-cutting document. No spec-change-requests were generated.
**Follow-ups**: 24 design-gate open questions to be resolved during Tier 3 specialist reviews or implementation.

---

## [2026-02-06 13:27] Architect — System Design Finalized (8 Entities, 14 Constraints)

**Changed**: Created system-design.yaml defining the full booking system architecture
**Decision**: Architecture uses Next.js App Router + Supabase (Postgres, Auth, RLS, Edge Functions) + Resend + Google Workspace. 8 entities, 14 key constraints, 7 flows, 7 risks documented. All 7 feature specs validated as consistent with the design — no spec-change-requests needed. Design gates carried forward: slot lock timeout value, service count limits, booking state machine transitions, and booking URL format (TBD).
**Follow-ups**: Design gates (slot lock timeout, URL format) remain open for resolution during implementation.

---

## [2026-02-06 13:28] Database Administrator — 8-Table Schema with Polymorphic Audit Log

**Changed**: Created db-migration-plan.yaml (10 phases, all expand/greenfield) and checks.yaml (db_migration: pass)
**Decision**: Schema uses 8 tables with FK dependencies and RLS policies. Key architecture decisions:
- **Services FK uses RESTRICT** — prevents deletion of services that have existing bookings, preserving referential integrity over convenience.
- **Audit log uses polymorphic pattern** — single audit_logs table with entity_type/entity_id columns rather than per-table audit tables, reducing schema complexity while maintaining full traceability.
- **Availability uses btree_gist exclusion constraint** — prevents overlapping availability windows at the database level rather than relying on application-level validation.
- **Slot locks use partial unique index** — ensures only one active lock per time slot without blocking expired/released locks.
- **Audit log immutability enforced** — no UPDATE/DELETE permitted on audit_logs table.
**Follow-ups**: Pre-deploy requirements: btree_gist extension must be enabled, column-level encryption for OAuth tokens, anon RLS routing must go through Edge Functions (not direct client access).

---

## [2026-02-06 13:34] Project Task Planner — 69 Tasks Across 7 Features

**Changed**: Created tasks.yaml for all 7 features (69 total tasks) plus task-order.md documenting implementation sequence
**Decision**: Full spec coverage verified — every requirement (R) and scenario (S) has an implements pointer in at least one task. 6 cross-feature dependency chains documented. Recommended implementation order: DB migrations -> Auth/Setup -> Core Entities (services, availability) -> Booking Flow (booking-page, booking-management) -> Integrations (google-calendar-sync) -> Notifications (booking-notifications). This order minimizes blocked work and ensures foundational schemas exist before dependent features.
**Follow-ups**: None. Tier 2 is complete. Tier 3 (ui-designer, security-engineer, compliance-engineer) can begin in parallel.

---

## [2026-02-06 14:00] UI Designer — 7 UI Surfaces Defined, Design Gates Resolved

**Changed**: Created ux-flows.md (735 lines) covering 7 UI surfaces: therapist dashboard, setup wizard, services management, availability management, public booking page, therapist booking management, client booking management (tokenized link). All screens include layout, component states (default/loading/empty/error), accessibility notes, responsive behavior, and transitions.
**Decision**: Resolved 5 previously open design-gate questions:
- **15-minute time increments** for availability and booking slots (balances flexibility with simplicity for solo therapists)
- **8-week availability window** for public booking page (sufficient forward visibility without overwhelming)
- **10-minute slot lock timeout** for booking reservations (long enough to complete booking, short enough to release abandoned slots)
- **Active/inactive service toggle** rather than delete (preserves booking history while hiding from public page)
- **Slug auto-generated from practice name** for booking URL (resolves URL format question — uses practice name slug)
All designs comply with .ops/ui-design-system.md (sage green palette, session card signature, warm consulting room feel). No spec-change-requests.
**Follow-ups**: Frontend designer (Tier 4) to produce component-level implementations from these UX flows.

---

## [2026-02-06 14:00] Security Engineer — Security Patterns and Threat Model Established

**Changed**: Created security-patterns.md. Updated checks.yaml with security section (PASS).
**Decision**: Key security architecture decisions:
- **Google OAuth with PKCE** and incremental authorization — tokens encrypted at rest, no refresh token in client
- **RLS policies defined for all 8 tables** — therapist data isolated by auth.uid(), public booking page uses anon role routed through Edge Functions
- **Zod validation** on all API boundaries (server actions, Edge Functions, webhook handlers)
- **Rate limiting** on public endpoints (booking page, slot lock acquisition)
- **STRIDE threat model** completed — spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege all addressed
- **PHI/PII classification** completed for all entities — booking data, therapist profiles, client contact info all classified with handling requirements
- **7 secrets inventoried** with storage and rotation requirements
5 design gates remain pending (calendar event title format, slot lock timeout confirmation, manage_token expiry, phone number format, max concurrent locks). 3 pre-deploy verifications required (OAuth token encryption, security headers, rate limiting infrastructure).
**Follow-ups**: Pre-deploy verifications must pass before production. 5 design gates to be resolved during implementation.

---

## [2026-02-06 14:00] Compliance Engineer — PHI-Adjacent Classification and Retention Policy

**Changed**: Created compliance-requirements.md. Updated checks.yaml with compliance section (PASS).
**Decision**: Key compliance decisions:
- **PHI-adjacent classification** for all booking data — not direct clinical notes, but appointment details with a healthcare provider constitute protected health information under PHIPA (Ontario) and PIPEDA
- **Column-by-column data classification** for all 8 tables — each column tagged with sensitivity level and handling requirements
- **10-year retention** for bookings and audit logs (PHIPA Ontario requirement); 2-year retention for notification logs
- **PIPEDA consent requirements** — unchecked-by-default checkbox with clear purpose language required at booking; no pre-checked consent boxes
- **BAA requirements identified** — Business Associate Agreements needed with Supabase, Resend (email), and Vercel before production launch
- 4 compliance-impacting design gates, 3 pre-production blockers (BAAs), 3 backlog items
**Follow-ups**: BAAs with Supabase, Resend, and Vercel are pre-production blockers. Data retention automation and consent tracking to be implemented in Tier 5.

---
