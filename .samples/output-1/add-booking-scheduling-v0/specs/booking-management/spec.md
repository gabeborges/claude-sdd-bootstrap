# Capability: Booking Management

Reschedule, cancellation, and manual booking capabilities.

## ADDED Requirements

### Requirement: Client Rescheduling

The system SHALL allow clients to reschedule their appointments.

#### Scenario: Reschedule via email link
- **WHEN** a client clicks the reschedule link in their confirmation or reminder email
- **THEN** they are taken to a reschedule page showing their current appointment
- **AND** can select a new available time slot

#### Scenario: Successful reschedule
- **WHEN** a client completes the reschedule flow
- **THEN** the original time slot becomes available again
- **AND** the new time slot is booked
- **AND** both client and therapist receive notification emails
- **AND** the Google Calendar event is updated

#### Scenario: Reschedule minimum notice
- **WHEN** a client attempts to reschedule
- **THEN** the minimum notice period applies to the new time selection
- **AND** they cannot reschedule to a time within the minimum notice window

### Requirement: Client Cancellation

The system SHALL allow clients to cancel their appointments.

#### Scenario: Cancel via email link
- **WHEN** a client clicks the cancel link in their confirmation or reminder email
- **THEN** they are taken to a cancellation page showing their appointment details
- **AND** must confirm the cancellation

#### Scenario: Successful cancellation
- **WHEN** a client confirms cancellation
- **THEN** the booking status changes to "cancelled"
- **AND** the time slot becomes available again
- **AND** both client and therapist receive notification emails
- **AND** the Google Calendar event is deleted

#### Scenario: Cancellation reason (optional)
- **WHEN** a client cancels an appointment
- **THEN** they can optionally provide a reason
- **AND** the reason is stored with the booking record

### Requirement: Therapist Rescheduling

The system SHALL allow therapists to reschedule bookings from their dashboard.

#### Scenario: Therapist initiates reschedule
- **WHEN** a therapist selects a booking to reschedule
- **THEN** they can select a new time from their available slots
- **AND** can provide a reason for the reschedule

#### Scenario: Client notification on therapist reschedule
- **WHEN** a therapist reschedules a booking
- **THEN** the client receives an email with the new appointment details
- **AND** the reason (if provided) is included in the email

### Requirement: Therapist Cancellation

The system SHALL allow therapists to cancel bookings from their dashboard.

#### Scenario: Therapist initiates cancellation
- **WHEN** a therapist cancels a booking
- **THEN** they must provide a reason for the cancellation
- **AND** the client receives a notification email with the reason

#### Scenario: Bulk cancellation
- **WHEN** a therapist needs to cancel multiple bookings (e.g., sick day)
- **THEN** they can select multiple bookings and cancel them together
- **AND** each client receives an individual notification email

### Requirement: Manual Booking Creation

The system SHALL allow therapists to create bookings manually.

#### Scenario: Create booking for existing client
- **WHEN** a therapist creates a manual booking
- **THEN** they can search and select from existing clients
- **AND** select a service, date, and time
- **AND** the booking is created with standard confirmation flow

#### Scenario: Create booking for new client
- **WHEN** a therapist creates a manual booking for a new client
- **THEN** they enter client details (name, email, phone)
- **AND** a new client record is created
- **AND** the client receives a confirmation email

#### Scenario: Override availability
- **WHEN** a therapist creates a manual booking
- **THEN** they can book outside their normal availability hours
- **AND** the system warns them but allows the booking

### Requirement: Booking Status Lifecycle

The system SHALL track booking status through defined states.

#### Scenario: Status transitions
- **WHEN** a booking is created
- **THEN** its status is "confirmed"
- **AND** can transition to "completed", "cancelled", or "no-show"

#### Scenario: Mark as completed
- **WHEN** the appointment time passes
- **THEN** the therapist can mark the booking as "completed"
- **OR** mark it as "no-show" if the client did not attend

#### Scenario: Past booking protection
- **WHEN** a booking's scheduled time has passed
- **THEN** the booking cannot be rescheduled
- **AND** cancellation is recorded but does not free up the slot

### Requirement: Therapist Booking Dashboard

The system SHALL provide therapists with a view of their bookings.

#### Scenario: View upcoming bookings
- **WHEN** a therapist views their dashboard
- **THEN** they see a list of upcoming bookings with date, time, client name, and service

#### Scenario: View booking details
- **WHEN** a therapist selects a booking
- **THEN** they see full details: client contact info, service, notes, status, and action buttons

#### Scenario: Calendar view
- **WHEN** a therapist views bookings in calendar view
- **THEN** bookings are displayed on a weekly/monthly calendar
- **AND** availability blocks are also shown

#### Scenario: Filter and search
- **WHEN** a therapist has many bookings
- **THEN** they can filter by date range, status, or service
- **AND** search by client name or email
