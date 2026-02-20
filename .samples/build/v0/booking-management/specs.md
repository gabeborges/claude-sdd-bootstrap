# Booking Management -- Specs

## Goal

Allow both therapists and clients to manage existing bookings -- including rescheduling and cancellation -- and allow therapists to manually create bookings on behalf of clients. This reduces back-and-forth communication and gives therapists control over their schedule.

## Non-Goals

- No booking policies (configurable cancellation/rescheduling rules such as deadlines or fees are deferred)
- No recurring appointment management
- No group session management
- No payment or refund handling tied to booking changes
- No client accounts or authenticated client portal (clients manage bookings via tokenized links)
- No booking history or analytics dashboard (v0)
- No waitlist or auto-fill for cancelled slots
- No AI-assisted booking management
- Not a full calendar management tool -- only booking lifecycle operations

## Requirements

- R1: Therapists must be able to view all their upcoming and past bookings.
- R2: Therapists must be able to reschedule a booking to a new available time slot. The new slot must pass the same availability and double-booking checks as a new booking.
- R3: Therapists must be able to cancel a booking. Cancellation must record the cancellation reason (optional) and update the booking status.
- R4: Therapists must be able to manually create a booking on behalf of a client by selecting a service, time slot, and entering client details (name, email, phone).
- R5: Clients must be able to reschedule their own booking. The mechanism for client access must not require a client account (e.g., tokenized link in confirmation email). (Requires design gate: define client access mechanism.)
- R6: Clients must be able to cancel their own booking via the same access mechanism.
- R7: When a booking is rescheduled, the original time slot must be released and become available for other bookings.
- R8: When a booking is cancelled, the time slot must be released and become available for other bookings.
- R9: All booking state changes (create, reschedule, cancel) must emit audit log entries (actor, action, timestamp, booking ID, before/after state).
- R10: Supabase RLS must enforce that therapists can only view and manage bookings within their own tenant. A therapist must not be able to see or modify another therapist's bookings.
- R11: Client access to booking management (via token) must be scoped to only the specific booking referenced by the token. Clients must not be able to access other bookings or therapist data.
- R12: Booking data must be encrypted at rest and in transit (TLS 1.2+).
- R13: All booking data must be stored in Canada-region infrastructure.
- R14: The booking management UI (therapist side) must be WCAG compliant.
- R15: Rescheduling must trigger the same slot-locking mechanism as new bookings to prevent double-booking during the rescheduling process.
- R16: Booking state transitions must be clearly defined: confirmed -> rescheduled, confirmed -> cancelled. (Requires design gate: define full state machine.)
- R17: When a therapist manually creates a booking, the same client data requirements apply (name, email, phone, consent). The therapist is the actor in the audit log for manual bookings.

## Acceptance Criteria (Scenarios)

### S1: Therapist views their bookings
**Given** a therapist has 5 upcoming bookings and 3 past bookings
**When** they navigate to their booking management page
**Then** all bookings are displayed with service name, client name, date/time, and status

### S2: Therapist reschedules a booking
**Given** a therapist has a booking "Initial Consultation with Jane Smith" on Monday at 10:00 AM
**When** the therapist selects reschedule and picks Wednesday at 2:00 PM (an available slot)
**Then** the booking is updated to Wednesday 2:00 PM, Monday 10:00 AM becomes available again, and an audit log entry is recorded with before/after times

### S3: Therapist cancels a booking
**Given** a therapist has a booking "Follow-up with John Doe" on Tuesday at 3:00 PM
**When** the therapist cancels the booking
**Then** the booking status changes to "cancelled", Tuesday 3:00 PM becomes available again, and an audit log entry is recorded

### S4: Therapist manually creates a booking
**Given** a therapist is on their booking management page
**When** they select "Create Booking", choose service "Follow-up Session", select Wednesday at 11:00 AM, enter client details (name, email, phone), confirm consent, and submit
**Then** the booking is created, the slot is reserved, and an audit log entry records the therapist as the actor

### S5: Client reschedules via tokenized link
**Given** a client received a confirmation email with a manage-booking link
**When** the client clicks the link and selects "Reschedule", then picks a new available slot
**Then** the booking is updated to the new time, the old slot is released, and an audit log entry is recorded with actor "client:{email}"

### S6: Client cancels via tokenized link
**Given** a client received a confirmation email with a manage-booking link
**When** the client clicks the link and selects "Cancel"
**Then** the booking status changes to "cancelled", the slot is released, and an audit log entry is recorded

### S7: Client token is scoped to single booking
**Given** a client has a manage-booking token for booking #123
**When** the client attempts to modify the token URL to access booking #456
**Then** the request is denied and the client sees an error or is redirected to their original booking only

### S8: Rescheduling prevents double-booking
**Given** a therapist is rescheduling a booking to Thursday at 9:00 AM
**When** another booking already exists at Thursday 9:00 AM
**Then** the system rejects the reschedule with a clear error indicating the slot is unavailable

### S9: Tenant isolation on booking management
**Given** Therapist A has bookings in their account
**When** Therapist B attempts to access Therapist A's bookings via API
**Then** the request is denied by RLS and no data is returned

### S10: Cancelled booking slot becomes available
**Given** a booking on Friday at 1:00 PM is cancelled
**When** a new client views the booking page for Friday
**Then** the 1:00 PM slot is shown as available

## Open Questions

- Q1: What is the client booking management access mechanism? Tokenized link in email is assumed, but the exact implementation (JWT, short-lived token, signed URL) needs to be determined. (Requires design gate.)
- Q2: Should there be any time restrictions on client self-service rescheduling/cancellation (e.g., "cannot cancel within 24 hours of appointment")? The PRD defers booking policies, but some minimal guard may be needed. (Requires design gate.)
- Q3: What is the full booking state machine? Possible states: pending, confirmed, rescheduled, cancelled, completed, no-show. Which states are in v0 scope? (Requires design gate.)
- Q4: Should the therapist see a reason field when cancelling, or is cancellation always without reason in v0?
