# Capability: Booking

Public booking page and client booking flow.

## ADDED Requirements

### Requirement: Public Booking Page

The system SHALL provide a public booking page for each therapist.

#### Scenario: Access booking page
- **WHEN** a client visits curio.health/{username}/booking
- **THEN** they see the therapist's practice name, available services, and can proceed to book

#### Scenario: Booking page performance
- **WHEN** a client loads the booking page
- **THEN** the page renders within 0.2 seconds

#### Scenario: Inactive therapist
- **WHEN** a client visits a booking page for an incomplete setup
- **THEN** they see a "coming soon" message
- **AND** cannot proceed with booking

#### Scenario: Invalid booking URL
- **WHEN** a client visits a non-existent booking page URL
- **THEN** they see a friendly 404 page with guidance

### Requirement: Service Selection

The system SHALL allow clients to select a service to book.

#### Scenario: View available services
- **WHEN** a client views the booking page
- **THEN** they see all visible services with name, duration, and description
- **AND** services appear in the therapist's configured order

#### Scenario: Select a service
- **WHEN** a client selects a service
- **THEN** they proceed to the date/time selection step
- **AND** see the service duration and any location options

#### Scenario: Service with location choice
- **WHEN** a client selects a service configured for "both" online and in-person
- **THEN** they must choose their preferred format before proceeding

### Requirement: Date and Time Selection

The system SHALL allow clients to select an available date and time.

#### Scenario: View available dates
- **WHEN** a client proceeds to date selection
- **THEN** they see a calendar view with available dates highlighted
- **AND** dates with no availability are not selectable

#### Scenario: View available times
- **WHEN** a client selects a date
- **THEN** they see all available time slots for that date
- **AND** times are displayed in the therapist's timezone

#### Scenario: Timezone display
- **WHEN** a client views available times
- **THEN** the therapist's timezone is clearly displayed (e.g., "Times shown in Eastern Time")

### Requirement: Slot Locking

The system SHALL prevent double-booking through slot locking.

#### Scenario: Lock slot on selection
- **WHEN** a client selects a time slot
- **THEN** the slot is temporarily locked for that client
- **AND** other clients see the slot as unavailable

#### Scenario: Lock expiration
- **WHEN** a client does not complete booking within 10 minutes
- **THEN** the slot lock expires
- **AND** the slot becomes available again

#### Scenario: Concurrent booking attempt
- **WHEN** two clients try to book the same slot simultaneously
- **THEN** only the first client to lock the slot can proceed
- **AND** the second client is informed the slot is no longer available

### Requirement: Client Information Collection

The system SHALL collect required client information during booking.

#### Scenario: Required fields
- **WHEN** a client proceeds to the information step
- **THEN** they must provide: full name, email address, and phone number
- **AND** must accept the consent checkbox

#### Scenario: Consent requirement
- **WHEN** a client attempts to complete booking
- **THEN** they must explicitly consent to storing their personal information
- **AND** cannot proceed without consent

#### Scenario: Email validation
- **WHEN** a client enters their email address
- **THEN** the system validates the email format
- **AND** prevents submission with invalid emails

#### Scenario: Returning client recognition
- **WHEN** a client enters an email that matches an existing client record
- **THEN** the system pre-fills their name and phone number
- **AND** allows them to update the information if needed

### Requirement: Booking Confirmation

The system SHALL confirm successful bookings to the client.

#### Scenario: Successful booking
- **WHEN** a client completes the booking flow
- **THEN** they see a confirmation page with appointment details
- **AND** receive a confirmation email
- **AND** the slot lock is released

#### Scenario: Confirmation details
- **WHEN** a booking is confirmed
- **THEN** the confirmation shows: date, time, service name, duration, location/Meet link (if online), and therapist name

#### Scenario: Add to calendar option
- **WHEN** a client views the confirmation page
- **THEN** they can download an ICS file to add the appointment to their calendar

### Requirement: Accessibility

The system SHALL meet WCAG accessibility standards.

#### Scenario: Screen reader compatibility
- **WHEN** a client uses a screen reader
- **THEN** all booking page elements have appropriate ARIA labels
- **AND** the booking flow is fully navigable

#### Scenario: Keyboard navigation
- **WHEN** a client navigates using only a keyboard
- **THEN** all interactive elements are focusable and operable
- **AND** focus order follows logical flow
