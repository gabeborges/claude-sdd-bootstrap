# Booking Page -- Specs

## Goal

Provide a public-facing booking page where clients can view a therapist's available services, select a time slot, provide their details, and complete a booking -- all without requiring a client account or login. This is the primary interface for client self-service booking and must eliminate back-and-forth scheduling communication.

## Non-Goals

- No client accounts or client login (clients book as guests)
- No payment or deposit collection at booking time (v0 is free)
- No intake forms attached to booking
- No group session booking
- No recurring appointment booking
- No branding customization or design polish (v0 constraint)
- No booking policies displayed (cancellation/rescheduling rules are deferred)
- No client-side calendar integration (e.g., "Add to my calendar" button)
- No mobile app -- web only
- No offline functionality
- Not a general-purpose scheduling widget -- this serves therapist booking only
- No AI-assisted booking suggestions

## Requirements

- R1: The booking page must be publicly accessible at a URL in the format `www.curio.health/{therapist-identifier}/booking` (exact URL structure requires design gate -- see Open Questions).
- R2: The booking page must display the therapist's active services (name, duration, modality, description).
- R3: After selecting a service, the booking page must display available time slots computed from the therapist's availability minus existing bookings and locked slots.
- R4: Time slots must be displayed in the therapist's timezone.
- R5: When a client selects a time slot, the slot must be locked/held to prevent double-booking. The lock must have a configurable timeout after which the slot is released if booking is not completed (requires design gate: define timeout duration).
- R6: To complete a booking, the client must provide: full name, email address, phone number, and explicit consent to store their personal information.
- R7: All four client fields (name, email, phone, consent) are required. The form must validate all fields before submission.
- R8: Email must be validated for format correctness. Phone number must be validated for reasonable format (requires design gate: define phone format rules -- Canadian numbers only? International?).
- R9: Consent must be an explicit, affirmative action (e.g., checkbox that is unchecked by default). The consent language must clearly state what data is being collected and for what purpose.
- R10: Upon successful booking, the client must see a confirmation screen with booking details (service name, date, time, therapist name).
- R11: The booking page must load in approximately 0.2 seconds (performance target from PRD).
- R12: The booking page must be WCAG compliant (keyboard navigable, screen reader accessible, sufficient color contrast, no purple/blue color palette per PRD design constraint).
- R13: Booking data (client name, email, phone, consent, appointment details) is PHI-adjacent and must be encrypted at rest and in transit (TLS 1.2+).
- R14: All booking data must be stored in Canada-region infrastructure.
- R15: The booking page must not expose any therapist data beyond what is necessary for booking (data minimization). Specifically: no therapist email, phone, or internal identifiers.
- R16: Supabase RLS must ensure that booking records are scoped to the correct therapist tenant.
- R17: A booking creation must emit an audit log entry (actor: "client:{email}", action: "booking.created", timestamp, booking details).
- R18: The booking page must gracefully handle edge cases: no services configured, no availability set, all slots booked.
- R19: The slot lock mechanism must prevent race conditions where two clients attempt to book the same slot simultaneously. Only one booking must succeed; the other must receive a clear error and be prompted to select a different time.
- R20: The booking page must not require JavaScript to display initial content (progressive enhancement preferred for accessibility and performance), or at minimum must function correctly with JavaScript enabled in modern evergreen browsers.

## Acceptance Criteria (Scenarios)

### S1: Client views available services
**Given** a therapist has configured 2 active services ("Initial Consultation - 60 min" and "Follow-up - 45 min")
**When** a client navigates to the therapist's booking page URL
**Then** both services are displayed with their name, duration, and description

### S2: Client selects a service and sees available time slots
**Given** a client is on the booking page and selects "Initial Consultation - 60 min"
**When** the available time slots load
**Then** only genuinely available 60-minute blocks are shown, in the therapist's timezone, excluding already-booked times

### S3: Client selects a slot and it becomes locked
**Given** a client selects the 10:00 AM slot on Monday
**When** the slot is selected
**Then** the slot is locked for that client and other clients viewing the page no longer see 10:00 AM as available

### S4: Slot lock times out and releases
**Given** a client has locked the 10:00 AM slot but has not completed the booking
**When** the lock timeout period expires
**Then** the 10:00 AM slot becomes available again for other clients

### S5: Client completes booking with valid details
**Given** a client has selected a slot and is on the booking details form
**When** they enter name ("Jane Smith"), email ("jane@example.com"), phone ("416-555-1234"), check the consent box, and submit
**Then** the booking is created, the confirmation screen is shown with full booking details, and an audit log entry is recorded

### S6: Client submits without consent
**Given** a client has filled in name, email, and phone but has NOT checked the consent checkbox
**When** they attempt to submit
**Then** the form displays a validation error on the consent field and the booking is not created

### S7: Client submits with invalid email
**Given** a client enters "not-an-email" in the email field
**When** they attempt to submit
**Then** the form displays a validation error on the email field

### S8: Double-booking prevention (race condition)
**Given** two clients (Client A and Client B) are both viewing the 2:00 PM slot
**When** Client A locks and completes the booking for 2:00 PM, and Client B attempts to book the same slot
**Then** Client A's booking succeeds; Client B receives a clear error message indicating the slot is no longer available and is prompted to choose another time

### S9: Booking page with no services configured
**Given** a therapist has not yet configured any services
**When** a client navigates to the therapist's booking page
**Then** the page displays a clear message indicating no services are currently available (not an error page)

### S10: Booking page with no availability
**Given** a therapist has services but has not set any availability
**When** a client navigates to the booking page and selects a service
**Then** the page displays a clear message indicating no time slots are currently available

### S11: Booking page performance
**Given** a therapist has services and availability configured
**When** a client navigates to the booking page on a modern browser with reasonable network conditions
**Then** the page loads and displays services within approximately 0.2 seconds

### S12: Booking page accessibility
**Given** a client is using a screen reader
**When** they navigate the booking page, select a service, choose a time slot, fill in details, and submit
**Then** the entire booking flow is fully accessible via keyboard and screen reader, with appropriate ARIA labels and focus management

### S13: Data minimization on booking page
**Given** a client views the booking page
**When** they inspect the page source or network requests
**Then** no therapist email, phone number, internal IDs, or unnecessary personal data is exposed

## Open Questions

- Q1: What is the exact booking URL format? PRD mentions `www.curio.health/{therapist-identifier}/booking` but also notes this is TBD. Is the identifier a username, slug, or UUID? (Requires design gate.)
- Q2: What is the slot lock timeout duration? Common values are 5-15 minutes. (Requires design gate.)
- Q3: What phone number format should be accepted? Canadian numbers only (10-digit), or international format? (Requires design gate.)
- Q4: Should the consent text be configurable by the therapist, or is it a standard system-provided consent statement? (Requires design gate.)
- Q5: Should the booking page show the timezone label to the client (e.g., "All times shown in Eastern Time")? (Requires design gate -- also raised in availability spec.)
