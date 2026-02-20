## ADDED Requirements

### Requirement: Public booking page at path-based URL
The system SHALL serve a public booking page at `/{slug}/bookings` where `{slug}` is the therapist's unique identifier. This page SHALL be accessible without authentication.

#### Scenario: Client visits a valid booking page
- **WHEN** a client navigates to `/{slug}/bookings` where the slug belongs to a registered therapist
- **THEN** the system displays the therapist's name and available booking slots

#### Scenario: Client visits a booking page with an invalid slug
- **WHEN** a client navigates to `/{slug}/bookings` where the slug does not match any therapist
- **THEN** the system displays a 404 page indicating the booking page was not found

### Requirement: Display available time slots
The booking page SHALL display available time slots based on the therapist's configured availability minus their existing Google Calendar events. All times SHALL be displayed in the therapist's configured time zone.

#### Scenario: Therapist has available slots
- **WHEN** a client views the booking page for a therapist with configured availability and open calendar time
- **THEN** the system displays selectable time slots for each available date, shown in the therapist's time zone

#### Scenario: Therapist has no available slots for a date
- **WHEN** a client views the booking page and the therapist's calendar is fully booked or no availability is configured for a given date
- **THEN** the system shows no available slots for that date

#### Scenario: Therapist's Google Calendar is unavailable
- **WHEN** the system cannot reach the Google Calendar API to compute availability
- **THEN** the booking page displays an error message indicating booking is temporarily unavailable

### Requirement: Client booking form
The booking page SHALL collect the client's name, email, and phone number when they select a time slot and submit a booking.

#### Scenario: Client submits a valid booking
- **WHEN** a client selects an available time slot and submits the form with a valid name, email, and phone number
- **THEN** the system creates a session, creates or links a client record, creates a Google Calendar event, and displays a booking confirmation

#### Scenario: Client submits incomplete information
- **WHEN** a client attempts to submit the booking form with missing required fields (name, email, or phone)
- **THEN** the system displays validation errors and does not create the booking

#### Scenario: Selected time slot is no longer available
- **WHEN** a client submits a booking for a time slot that was booked by someone else between page load and submission
- **THEN** the system displays an error indicating the slot is no longer available and refreshes the available slots

### Requirement: Booking confirmation display
After a successful booking, the system SHALL display a confirmation with the session date, time (in therapist's time zone), and a cancellation link.

#### Scenario: Successful booking confirmation
- **WHEN** a booking is successfully created
- **THEN** the system displays the session details and provides a unique cancellation link the client can use to cancel
