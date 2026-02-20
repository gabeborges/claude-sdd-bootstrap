# Design: Booking & Scheduling System (v0)

## Context

Curio v0 establishes the foundational practice management wedge for solo online therapists. This is a greenfield implementation that must:
- Integrate deeply with Google Workspace (Calendar, Meet, OAuth)
- Meet HIPAA/PHIPA compliance requirements from day 0
- Establish patterns that future verticals (notes, billing, communication) will build upon
- Serve Canadian therapists only (Canada data residency)

**Stakeholders:** Solo online therapists (primary users), their clients (booking page users)

## Goals / Non-Goals

**Goals:**
- Clients can self-book without back-and-forth communication
- Therapists spend minimal time on booking administration
- Establish foundational compliance patterns (audit logging, policy checks)
- One-way Google Calendar sync with Meet link generation
- Fast, accessible public booking experience

**Non-Goals:**
- Two-way calendar sync (read from Google Calendar)
- Payment processing
- Client accounts or authentication
- Multi-therapist/practice support
- Recurring appointments or group sessions

## Decisions

### 1. Data Architecture

**Decision:** Supabase as single database with row-level security (RLS) for tenant isolation

**Rationale:**
- Matches tech stack from product vision
- RLS provides compliance-friendly access control at the database level
- Supabase Auth integrates well with Google OAuth
- Canada region available for data residency

**Core Entities:**
- `therapists` - Profile, timezone, settings (linked to Google identity)
- `services` - Appointment types with duration, price, location type
- `availability_schedules` - Weekly recurring availability patterns
- `availability_overrides` - Date-specific exceptions (blocked/available)
- `bookings` - Individual appointments with status lifecycle
- `clients` - Minimal client records (name, email, phone, consent)
- `audit_logs` - Immutable event log for compliance

### 2. Google Workspace Integration

**Decision:** One-way sync (Curio → Google Calendar) with explicit user actions

**Rationale:**
- Per product vision: "no silent writes" to Workspace
- Simplifies conflict resolution (Curio is source of truth for bookings)
- Calendar events are created on booking confirmation, updated on reschedule, deleted on cancellation
- Google Meet links generated via Calendar API (conferenceDataVersion=1)

**OAuth Scopes (minimal):**
- `openid`, `email`, `profile` - Identity
- `https://www.googleapis.com/auth/calendar.events` - Create/update/delete events
- Future: `calendar.readonly` for availability checking (not in v0)

### 3. Booking Flow Architecture

**Decision:** Slot locking with TTL to prevent double-booking

**Rationale:**
- Race condition prevention is critical for user experience
- Simple implementation: lock slot in database with 10-minute TTL
- If booking not completed, slot auto-releases
- No need for complex distributed locking in v0 scale

**Flow:**
1. Client views available slots (computed from availability - existing bookings)
2. Client selects slot → slot locked with TTL
3. Client provides details → booking created, slot lock released
4. Calendar event created → confirmation email sent

### 4. Notification System

**Decision:** Resend for transactional emails with Supabase scheduled jobs for reminders

**Rationale:**
- Resend matches tech stack, good deliverability
- Supabase pg_cron for scheduling reminder jobs (day before, 1 hour before)
- Simple, no external job queue needed for v0 scale

**Email Types:**
- Booking confirmation (immediate)
- Booking rescheduled (immediate)
- Booking cancelled (immediate)
- Reminder - day before (scheduled)
- Reminder - 1 hour before (scheduled)

### 5. Public Booking Page

**Decision:** Next.js static generation with ISR for booking pages

**Rationale:**
- Fast load times (~0.2s target)
- SEO-friendly for therapist discovery
- ISR (Incremental Static Regeneration) for availability updates
- URL format: `curio.health/{username}/booking`

**Client-side:**
- Availability fetched dynamically (real-time accuracy)
- Booking submission via API route
- No client authentication required

### 6. Audit Logging Pattern

**Decision:** Append-only audit_logs table with structured events

**Rationale:**
- HIPAA/PHIPA requires audit trail for PHI access
- Simple, queryable, exportable
- Every booking action logged: create, view, update, cancel
- Actor (therapist/client/system), action, resource, timestamp, metadata

**Event Structure:**
```
{
  actor_type: 'therapist' | 'client' | 'system',
  actor_id: string,
  action: string,
  resource_type: string,
  resource_id: string,
  metadata: jsonb,
  created_at: timestamp
}
```

### 7. Timezone Handling

**Decision:** Store all times in UTC, therapist timezone for display

**Rationale:**
- Avoids timezone bugs in scheduling logic
- Therapist sets timezone during onboarding
- Booking page displays in therapist's timezone (per PRD requirement)
- Database stores UTC, application converts for display

## Alternatives Considered

### Two-way Calendar Sync
- **Rejected:** Adds complexity (conflict resolution), not required for v0 goals
- **Future consideration:** When therapists want existing calendar events to block availability

### Client Accounts
- **Rejected:** Friction for booking, not needed for v0
- **Future consideration:** For returning clients, appointment history, preferences

### External Job Queue (BullMQ, etc.)
- **Rejected:** Overkill for v0 scale, adds infrastructure complexity
- **Future consideration:** If reminder volume exceeds pg_cron limits

## Risks / Trade-offs

| Risk | Impact | Mitigation |
|------|--------|------------|
| Slot lock expiry during slow booking | Poor UX, lost booking | 10-min TTL is generous; extend if needed |
| Google API rate limits | Failed calendar sync | Implement retry with backoff; monitor usage |
| Reminder email delivery | Missed appointments | Resend has good deliverability; add SMS later |
| Timezone bugs | Wrong appointment times | Comprehensive testing; use proven libraries (date-fns-tz) |

## Migration Plan

Not applicable - greenfield implementation.

## Open Questions

1. **Booking URL format** - `curio.health/{username}/booking` vs `curio.health/book/{id}`? Leaning toward username for branding.
2. **Minimum notice default** - What's the sensible default for minimum booking lead time? Suggest 2 hours.
3. **Slot lock duration** - 10 minutes sufficient? May need tuning based on real usage.
