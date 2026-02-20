# Google Calendar Sync -- Specs

## Goal

Synchronize bookings from Curio to the therapist's Google Calendar as calendar events, and generate Google Meet links for online appointments. This ensures the therapist's Google Calendar reflects their Curio bookings and provides video conferencing for online sessions -- all without requiring the therapist to manually create calendar events.

## Non-Goals

- No two-way sync (reading from Google Calendar to Curio is out of scope -- sync is one-way: Curio to GCal only)
- No reading Google Calendar to detect conflicts or block availability
- No Google Calendar as source of truth for availability (Curio's availability config is authoritative)
- No name display format selection for calendar events (deferred nice-to-have)
- No multiple Google Calendar support (one calendar per therapist)
- No non-Google calendar integration (Google Workspace only per product vision)
- No silent writes -- every calendar write must correspond to an explicit booking action and be audit-logged
- No AI-driven calendar management
- No offline calendar sync or queue-based deferred sync beyond retry for transient failures

## Requirements

- R1: When a booking is created in Curio, a corresponding Google Calendar event must be created in the therapist's Google Calendar.
- R2: When a booking is rescheduled in Curio, the corresponding Google Calendar event must be updated to reflect the new date/time.
- R3: When a booking is cancelled in Curio, the corresponding Google Calendar event must be deleted or marked as cancelled in Google Calendar.
- R4: For bookings with modality "online", the Google Calendar event must include an automatically generated Google Meet link.
- R5: For bookings with modality "in-person", no Google Meet link must be generated.
- R6: The Google Calendar event must include relevant booking details: service name, client name, appointment time, and duration. (Requires design gate: define exact event title and description format to avoid exposing excessive PHI in calendar.)
- R7: Curio must store a reference to the Google Calendar event (event ID) for each booking, to enable updates and deletions. Curio must not store a full duplicate of the calendar event data.
- R8: Google Calendar sync must use the therapist's OAuth credentials obtained during setup. OAuth scopes must follow least-privilege principle (only calendar event creation/modification, not full calendar read access unless required by API).
- R9: Every Google Calendar write (create, update, delete) must be audit-logged (actor: system, action: "gcal.event.created/updated/deleted", timestamp, booking reference, event ID).
- R10: No silent writes -- calendar events must only be created/modified/deleted as a direct result of a booking action (create, reschedule, cancel). The system must never autonomously modify the therapist's calendar outside of booking operations.
- R11: Google Calendar API failures must not block or roll back the booking. The booking is the source of truth. Calendar sync failures must be logged and retried. (Requires design gate: define retry strategy and failure notification.)
- R12: All OAuth tokens and Google API credentials must be encrypted at rest and stored in Canada-region infrastructure.
- R13: OAuth token refresh must be handled automatically. If a refresh token becomes invalid (e.g., therapist revokes access), the system must gracefully degrade and notify the therapist that calendar sync is disconnected.
- R14: Calendar events must be created in the therapist's timezone.
- R15: Supabase RLS must ensure that calendar sync operations are scoped to the correct therapist tenant. One therapist's sync must not affect another's calendar.

## Acceptance Criteria (Scenarios)

### S1: Calendar event created on new booking
**Given** a client books "Initial Consultation" for Monday at 10:00 AM with Therapist Dr. Smith
**When** the booking is confirmed
**Then** a Google Calendar event is created in Dr. Smith's Google Calendar for Monday 10:00 AM - 11:00 AM with the service name and client name, and an audit log entry is recorded

### S2: Online booking includes Meet link
**Given** a client books an online service
**When** the Google Calendar event is created
**Then** the event includes an automatically generated Google Meet link, and the Meet link is stored in the booking record for inclusion in notification emails

### S3: In-person booking has no Meet link
**Given** a client books an in-person service
**When** the Google Calendar event is created
**Then** the event does not include a Google Meet link

### S4: Calendar event updated on reschedule
**Given** a booking has a corresponding Google Calendar event on Monday 10:00 AM
**When** the booking is rescheduled to Wednesday 2:00 PM
**Then** the Google Calendar event is updated to Wednesday 2:00 PM - 3:00 PM and an audit log entry is recorded

### S5: Calendar event removed on cancellation
**Given** a booking has a corresponding Google Calendar event
**When** the booking is cancelled
**Then** the Google Calendar event is deleted (or marked cancelled) and an audit log entry is recorded

### S6: Calendar API failure does not block booking
**Given** a client completes a booking but the Google Calendar API is temporarily unavailable
**When** the booking is confirmed
**Then** the booking is saved and confirmed in Curio, the calendar sync failure is logged, and the system retries the calendar event creation

### S7: OAuth token refresh
**Given** a therapist's OAuth access token has expired
**When** a booking is created and calendar sync is triggered
**Then** the system automatically refreshes the token using the refresh token and proceeds with the calendar event creation

### S8: OAuth revocation handling
**Given** a therapist has revoked Curio's Google Calendar access
**When** a booking is created and calendar sync is triggered
**Then** the sync fails gracefully, the booking remains valid, and the therapist is notified (via UI or email) that Google Calendar sync is disconnected and needs to be re-authorized

### S9: No silent writes to Google Calendar
**Given** no booking action has occurred
**When** the system runs any background process
**Then** no Google Calendar events are created, modified, or deleted -- calendar writes happen exclusively as a result of booking create/reschedule/cancel actions

### S10: Tenant isolation for calendar sync
**Given** Therapist A and Therapist B both have connected Google Calendars
**When** a booking is created for Therapist A
**Then** the calendar event is created only in Therapist A's Google Calendar; Therapist B's calendar is not affected

## Open Questions

- Q1: What should the Google Calendar event title and description contain? Including the client's full name may be a PHI concern. Should it use initials, first name only, or a generic title like "Client Appointment"? (Requires design gate -- impacts both this spec and booking-notifications.)
- Q2: What is the retry strategy for Google Calendar API failures? How many retries, what backoff interval, and what happens if retries are exhausted? (Requires design gate.)
- Q3: Which specific Google OAuth scopes are required? The minimum for calendar event CRUD and Meet link generation needs to be identified. (Requires design gate.)
- Q4: If a therapist disconnects and reconnects Google Calendar, should existing bookings be re-synced to their calendar? (Requires design gate.)
