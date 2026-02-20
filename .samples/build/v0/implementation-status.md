# v0 Implementation Status

> Living tracker for the v0 Booking & Scheduling build.
> Updated by context-manager as tiers complete and status changes.

**Feature:** Booking & Scheduling
**Version:** v0
**Target Users:** Solo online therapists (Canada only) and their clients
**Last Updated:** 2026-02-06 (Tier 3 complete)

---

## Overall Progress

| Phase | Status | Notes |
|-------|--------|-------|
| PRD | Complete | 7 must-have features defined. See `.ops/build/v0/prd.md` |
| Specs (spec-writer) | **Complete** | 7 specs, 115 requirements, 75 acceptance scenarios, 24 open questions |
| System Design (architect) | **Complete** | 8 entities, 14 constraints, 7 flows, 7 risks |
| DB Migration Plan (database-administrator) | **Complete** | 10 phases (all expand/greenfield), 8 tables, checks.yaml pass |
| Tasks (project-task-planner) | **Complete** | 69 tasks across 7 features, 6 cross-feature dependency chains |
| UI Design (ui-designer) | **Complete** | 7 UI surfaces, 735-line ux-flows.md, 5 design gates resolved |
| Security Review (security-engineer) | **Complete** | Security patterns, STRIDE threat model, checks.yaml security: PASS |
| Compliance Review (compliance-engineer) | **Complete** | PHI-adjacent classification, retention policy, checks.yaml compliance: PASS |
| Frontend Design (frontend-designer) | Pending | Tier 4 — unblocked (Tier 3 complete) |
| Implementation (fullstack-developer) | Pending | Tier 5 — blocked by Tier 4 |
| Test Automation (test-automator) | Pending | Tier 5 — blocked by Tier 4 |
| QA (qa) | Pending | Tier 6 — blocked by Tier 5 |
| Code Review (code-reviewer) | Pending | Tier 6 — blocked by Tier 5 |
| Security Audit (security-auditor) | Pending | Tier 6 — blocked by Tier 5 |
| Compliance Audit (compliance-auditor) | Pending | Tier 6 — blocked by Tier 5 |

---

## Must-Have Features (v0 Scope)

| # | Feature | Workspace | Design Status | Build Status |
|---|---------|-----------|--------------|--------------|
| 1 | Service/appointment CRUD | `booking-services/` | Spec + Tasks complete | Pending |
| 2 | Availability management | `availability/` | Spec + Tasks complete | Pending |
| 3 | Public booking page | `booking-page/` | Spec + Tasks complete | Pending |
| 4 | Booking management (reschedule/cancel) | `booking-management/` | Spec + Tasks complete | Pending |
| 5 | Booking emails & reminders | `booking-notifications/` | Spec + Tasks complete | Pending |
| 6 | Google Calendar sync (one-way) | `google-calendar-sync/` | Spec + Tasks complete | Pending |
| 7 | Therapist setup (auth, profile, onboarding) | `therapist-setup/` | Spec + Tasks complete | Pending |

---

## Key Constraints Tracked

- **HIPAA/PHIPA compliance from day 0** — all booking data is PHI-adjacent
- **Canada-only data residency** — all data stored in Canada
- **No silent writes** — Google Workspace modifications require explicit action + audit entry
- **Supabase RLS non-negotiable** — never bypass row-level security
- **WCAG accessibility required** — not optional

---

## Artifacts Produced

| Artifact | Path | Status |
|----------|------|--------|
| PRD | `.ops/build/v0/prd.md` | Complete |
| Specs (booking-services) | `.ops/build/v0/booking-services/specs.md` | Complete |
| Specs (availability) | `.ops/build/v0/availability/specs.md` | Complete |
| Specs (booking-page) | `.ops/build/v0/booking-page/specs.md` | Complete |
| Specs (booking-management) | `.ops/build/v0/booking-management/specs.md` | Complete |
| Specs (booking-notifications) | `.ops/build/v0/booking-notifications/specs.md` | Complete |
| Specs (google-calendar-sync) | `.ops/build/v0/google-calendar-sync/specs.md` | Complete |
| Specs (therapist-setup) | `.ops/build/v0/therapist-setup/specs.md` | Complete |
| System Design | `.ops/build/system-design.yaml` | Complete |
| DB Migration Plan | `.ops/build/v0/db-migration-plan.yaml` | Complete |
| Checks | `.ops/build/v0/checks.yaml` | Complete (db_migration: pass, security: pass, compliance: pass) |
| UX Flows | `.ops/build/v0/ux-flows.md` | Complete |
| Security Patterns | `.ops/build/v0/security-patterns.md` | Complete |
| Compliance Requirements | `.ops/build/v0/compliance-requirements.md` | Complete |
| Tasks (booking-services) | `.ops/build/v0/booking-services/tasks.yaml` | Complete |
| Tasks (availability) | `.ops/build/v0/availability/tasks.yaml` | Complete |
| Tasks (booking-page) | `.ops/build/v0/booking-page/tasks.yaml` | Complete |
| Tasks (booking-management) | `.ops/build/v0/booking-management/tasks.yaml` | Complete |
| Tasks (booking-notifications) | `.ops/build/v0/booking-notifications/tasks.yaml` | Complete |
| Tasks (google-calendar-sync) | `.ops/build/v0/google-calendar-sync/tasks.yaml` | Complete |
| Tasks (therapist-setup) | `.ops/build/v0/therapist-setup/tasks.yaml` | Complete |
| Task Order | `.ops/build/v0/task-order.md` | Complete |

---

## Tier 2 Key Decisions Summary

- **8-table schema**: services, availability_rules, availability_overrides, bookings, booking_locks, audit_logs, therapist_profiles, oauth_tokens
- **Services FK RESTRICT**: Cannot delete services with existing bookings
- **Polymorphic audit log**: Single table, entity_type/entity_id pattern
- **btree_gist exclusion**: Prevents overlapping availability at DB level
- **Slot locks with partial unique index**: One active lock per slot, expired locks ignored
- **69 tasks, 6 dependency chains**: Implementation order documented in task-order.md

---

## Open Questions (from PRD + Specs)

- ~~24 design-gate open questions from specs~~ — 5 resolved by UI designer, 5 pending from security, 4 pending from compliance; remaining resolved or carried to implementation
- ~~UX/Design not finalized for v0~~ — UX flows complete (7 surfaces, sage green palette, warm consulting room feel)
- ~~Booking URL structure TBD~~ — Resolved: slug auto-generated from practice name
- ~~Slot lock timeout value TBD~~ — Resolved: 10-minute timeout
- Branding customization scope undefined (carried forward)
- **Security design gates pending** (5): calendar event title format, slot lock timeout confirmation, manage_token expiry, phone number format, max concurrent locks
- **Compliance design gates pending** (4): compliance-impacting gates documented in compliance-requirements.md
- **Pre-production blockers** (3): BAAs required with Supabase, Resend, and Vercel

---

## Tier 3 Key Decisions Summary

- **15-minute time increments** for all scheduling (UI designer)
- **8-week availability window** for public booking (UI designer)
- **10-minute slot lock timeout** (UI designer, confirmed by security)
- **Active/inactive toggle** for services instead of delete (UI designer)
- **Slug from practice name** for booking URLs (UI designer)
- **Google OAuth with PKCE** + incremental authorization (security engineer)
- **RLS on all 8 tables** with anon routed through Edge Functions (security engineer)
- **STRIDE threat model** completed (security engineer)
- **PHI-adjacent classification** for all booking data (compliance engineer)
- **10-year retention** for bookings/audit logs, 2-year for notification logs (compliance engineer)
- **PIPEDA consent** — unchecked-by-default checkbox required (compliance engineer)
- **BAAs required** with Supabase, Resend, Vercel before production (compliance engineer)

---

## Next Phase

**Tier 4 (Frontend Design):** frontend-designer. Now unblocked — depends on ux-flows.md from Tier 3.
**Then Tier 5 (Implementation):** fullstack-developer and test-automator. Blocked by Tier 4 completion.

---

## Deviations

_None recorded._

---

## Blockers

_None at this time._
