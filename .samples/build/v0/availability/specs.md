# Availability -- Specs

## Goal

Allow therapists to define when they are available for bookings -- their working days, hours, and time blocks -- so the booking page can display only valid, bookable time slots to clients. This includes timezone selection and minimum notice configuration.

## Non-Goals

- No per-service availability (all services share the same availability schedule in v0)
- No date-specific overrides or exceptions (e.g., holidays, sick days)
- No buffer time between appointments (deferred nice-to-have)
- No recurring schedule templates or copying across weeks
- No availability sync from Google Calendar (one-way Curio to GCal only; no reading GCal to block slots)
- No multi-therapist shared availability
- No AI-assisted schedule optimization
- Not a full calendar management tool -- this is availability configuration only

## Requirements

- R1: Therapists must be able to select their timezone during setup. All availability times are stored and interpreted in the therapist's selected timezone.
- R2: Therapists must be able to set available days of the week (e.g., Monday through Friday, or any subset).
- R3: Therapists must be able to define multiple time blocks per day (e.g., 9:00 AM - 12:00 PM and 2:00 PM - 5:00 PM on Monday).
- R4: Therapists must be able to configure a minimum notice period for bookings (e.g., "no same-day bookings" or "at least 24 hours notice"). The minimum notice period must be expressed in hours.
- R5: Time blocks must not overlap within the same day. The system must validate and reject overlapping time blocks.
- R6: Availability data must belong to exactly one therapist (tenant isolation). Supabase RLS must enforce that a therapist can only view and modify their own availability.
- R7: All availability create, update, and delete operations must emit an audit log entry (actor, action, timestamp, affected record).
- R8: The booking page must compute available time slots by intersecting the therapist's availability with their existing bookings (and slot locks), producing only genuinely bookable slots.
- R9: Available time slots displayed on the booking page must be shown in the therapist's timezone.
- R10: Availability data must be encrypted at rest and in transit (TLS 1.2+).
- R11: All availability data must be stored in Canada-region infrastructure.
- R12: The availability management UI must be WCAG compliant.
- R13: Changes to availability must not affect existing confirmed bookings (only future slot availability).

## Acceptance Criteria (Scenarios)

### S1: Therapist sets weekly availability
**Given** a therapist is on the availability management page
**When** they select Monday, Wednesday, and Friday as available days and set hours 9:00 AM - 5:00 PM for each
**Then** the availability is saved and reflected on their booking page as bookable time slots for those days and hours

### S2: Therapist configures multiple time blocks per day
**Given** a therapist is editing Monday's availability
**When** they add two time blocks: 9:00 AM - 12:00 PM and 2:00 PM - 5:00 PM
**Then** both blocks are saved, and the booking page shows available slots only within those two blocks (not 12:00 PM - 2:00 PM)

### S3: Therapist sets timezone
**Given** a therapist is completing their setup
**When** they select "America/Toronto" as their timezone
**Then** all availability times are stored in that timezone, and the booking page displays slots in "America/Toronto"

### S4: Overlapping time blocks are rejected
**Given** a therapist is editing Tuesday's availability
**When** they attempt to add blocks 9:00 AM - 1:00 PM and 12:00 PM - 4:00 PM (overlapping)
**Then** the system rejects the configuration with a clear error message indicating the overlap

### S5: Minimum notice prevents same-day booking
**Given** a therapist has set minimum notice to 24 hours
**When** a client views the booking page at 10:00 AM on Monday
**Then** no slots are shown for Monday; the earliest available slots are on Tuesday (or later, depending on availability)

### S6: Minimum notice set to zero allows immediate booking
**Given** a therapist has set minimum notice to 0 hours
**When** a client views the booking page
**Then** slots from the current time onward (within today's remaining availability) are shown as bookable

### S7: Tenant isolation prevents cross-therapist access
**Given** Therapist A has configured their availability
**When** Therapist B attempts to view or modify Therapist A's availability
**Then** the request is denied by RLS policy

### S8: Existing bookings are unaffected by availability changes
**Given** a therapist has a confirmed booking on Wednesday at 3:00 PM
**When** the therapist removes Wednesday from their available days
**Then** the existing Wednesday booking remains confirmed and accessible; future Wednesdays no longer show available slots

### S9: Slot computation excludes already-booked times
**Given** a therapist is available Monday 9:00 AM - 5:00 PM with 60-minute services
**When** a client views Monday's availability and there is already a booking at 10:00 AM - 11:00 AM
**Then** the 10:00 AM slot is not shown as available; 9:00 AM, 11:00 AM, 12:00 PM, etc. are shown

### S10: Availability UI is accessible
**Given** a therapist is using keyboard-only navigation
**When** they interact with the availability configuration interface
**Then** all day selectors, time inputs, and save actions are reachable and operable via keyboard

## Open Questions

- Q1: Should the time block granularity be fixed (e.g., 15-minute increments) or free-form? This affects how slots are computed and displayed. (Requires design gate.)
- Q2: How far into the future should availability extend on the booking page? Indefinitely, or a configurable number of weeks? (Requires design gate.)
- Q3: Should the client-facing booking page also show the timezone label (e.g., "Times shown in ET") or just display times without explicit timezone indication? The PRD says "therapist's timezone" but does not specify client-facing label. (Requires design gate.)
