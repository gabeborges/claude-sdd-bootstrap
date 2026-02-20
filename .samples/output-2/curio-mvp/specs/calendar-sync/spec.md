## ADDED Requirements

### Requirement: Read therapist's Google Calendar for availability
The system SHALL query the Google Calendar FreeBusy API to determine the therapist's busy periods when computing available booking slots. Existing calendar events SHALL block those time periods from being bookable.

#### Scenario: Therapist has existing calendar events
- **WHEN** the system computes available slots for a therapist who has events on their Google Calendar
- **THEN** the time periods covered by those events are excluded from the available slots

#### Scenario: Therapist has no existing calendar events
- **WHEN** the system computes available slots for a therapist with a clear Google Calendar
- **THEN** all configured availability windows are shown as available slots

### Requirement: Create Google Calendar event on booking
The system SHALL create a Google Calendar event on the therapist's primary calendar when a client successfully books a session. The system SHALL store the Google Calendar event ID on the session record.

#### Scenario: Booking creates a calendar event
- **WHEN** a client successfully books a session
- **THEN** the system creates an event on the therapist's Google Calendar with the session date, time, and client name, and stores the event ID on the session record

#### Scenario: Calendar event creation fails
- **WHEN** the system attempts to create a Google Calendar event but the API call fails
- **THEN** the system rolls back the session creation and informs the client that booking failed

### Requirement: Delete Google Calendar event on cancellation
The system SHALL delete the corresponding Google Calendar event when a session is cancelled. The system SHALL use the stored event ID to identify the correct event.

#### Scenario: Cancellation removes calendar event
- **WHEN** a session with a stored Google Calendar event ID is cancelled
- **THEN** the system deletes the event from the therapist's Google Calendar

#### Scenario: Calendar event deletion fails
- **WHEN** the system attempts to delete a Google Calendar event but the API call fails (e.g., event already deleted manually)
- **THEN** the system still marks the session as cancelled in the database and logs the calendar sync failure

### Requirement: Prevent double-booking
The system SHALL enforce a unique constraint on (therapist_id, start_time) for confirmed sessions to prevent two clients from booking the same time slot.

#### Scenario: Simultaneous booking attempts for the same slot
- **WHEN** two clients attempt to book the same time slot concurrently
- **THEN** only one booking succeeds; the other receives an error that the slot is no longer available
