# Capability: Google Calendar Sync

One-way synchronization from Curio to Google Calendar.

## ADDED Requirements

### Requirement: Calendar Event Creation

The system SHALL create Google Calendar events when bookings are confirmed.

#### Scenario: Create event on booking
- **WHEN** a booking is successfully created
- **THEN** a Google Calendar event is created on the therapist's calendar
- **AND** the event includes: title, start/end time, description with client details

#### Scenario: Event title format
- **WHEN** creating a calendar event
- **THEN** the title follows the format: "[Service Name] - [Client Name]"
- **AND** does not expose sensitive information in the title

#### Scenario: Event description
- **WHEN** creating a calendar event
- **THEN** the description includes:
  - Client name, email, and phone
  - Service name and duration
  - Booking reference ID
  - Link to view booking in Curio

#### Scenario: Store event reference
- **WHEN** a calendar event is created
- **THEN** the Google Calendar event ID is stored with the booking
- **AND** enables future updates/deletion

### Requirement: Google Meet Integration

The system SHALL generate Google Meet links for online appointments.

#### Scenario: Meet link for online service
- **WHEN** a booking is created for an online service
- **THEN** the calendar event includes an automatically generated Google Meet link
- **AND** the Meet link is stored with the booking

#### Scenario: No Meet link for in-person
- **WHEN** a booking is created for an in-person service
- **THEN** no Google Meet link is generated
- **AND** the event location shows the practice address (if configured)

#### Scenario: Meet link in notifications
- **WHEN** a booking has a Meet link
- **THEN** the link is included in confirmation and reminder emails
- **AND** is prominently displayed for easy access

### Requirement: Calendar Event Updates

The system SHALL update Google Calendar events when bookings change.

#### Scenario: Update on reschedule
- **WHEN** a booking is rescheduled
- **THEN** the corresponding Google Calendar event is updated with the new time
- **AND** the Meet link (if any) is preserved

#### Scenario: Update on client info change
- **WHEN** client information is updated for a booking
- **THEN** the calendar event description is updated

### Requirement: Calendar Event Deletion

The system SHALL delete Google Calendar events when bookings are cancelled.

#### Scenario: Delete on cancellation
- **WHEN** a booking is cancelled
- **THEN** the corresponding Google Calendar event is deleted
- **AND** the event reference is cleared from the booking

#### Scenario: Handle missing event
- **WHEN** attempting to delete an event that no longer exists
- **THEN** the system handles the error gracefully
- **AND** logs the discrepancy for monitoring

### Requirement: Sync Reliability

The system SHALL ensure reliable calendar synchronization.

#### Scenario: Retry on failure
- **WHEN** a calendar API call fails
- **THEN** the system retries with exponential backoff
- **AND** logs the failure for monitoring

#### Scenario: Token refresh
- **WHEN** the OAuth token expires
- **THEN** the system automatically refreshes the token
- **AND** retries the failed operation

#### Scenario: Permission revocation handling
- **WHEN** a therapist revokes calendar permissions
- **THEN** the system detects the authorization error
- **AND** notifies the therapist to re-authorize
- **AND** bookings continue to work but without calendar sync

#### Scenario: Rate limit handling
- **WHEN** Google Calendar API rate limits are reached
- **THEN** the system queues pending sync operations
- **AND** processes them when limits reset

### Requirement: Audit Trail for Calendar Operations

The system SHALL log all calendar synchronization operations.

#### Scenario: Log calendar writes
- **WHEN** any calendar operation is performed (create, update, delete)
- **THEN** an audit log entry is created with:
  - Operation type
  - Booking ID
  - Calendar event ID
  - Success/failure status
  - Timestamp

#### Scenario: No silent writes
- **WHEN** creating or modifying calendar events
- **THEN** all operations are traceable to a user action or system event
- **AND** the triggering action is recorded
