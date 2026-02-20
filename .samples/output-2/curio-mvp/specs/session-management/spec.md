## ADDED Requirements

### Requirement: Session creation from booking
The system SHALL create a session record when a client successfully books a time slot. The session SHALL be linked to the therapist and the client, and SHALL store the start time, end time, status, and Google Calendar event ID.

#### Scenario: Session created on successful booking
- **WHEN** a client completes the booking form and the Google Calendar event is created
- **THEN** the system creates a session record with status `confirmed`, linked to the therapist and client

### Requirement: Session cancellation by therapist
The system SHALL allow therapists to cancel their own sessions from the dashboard. Cancellation SHALL set the session status to `cancelled` and trigger Google Calendar event deletion.

#### Scenario: Therapist cancels a session
- **WHEN** a therapist selects a confirmed session and confirms cancellation
- **THEN** the system sets the session status to `cancelled` and deletes the corresponding Google Calendar event

### Requirement: Session cancellation by client via token link
The system SHALL generate a unique cancellation token for each session. Clients SHALL be able to cancel a session by visiting a cancellation URL containing this token. No authentication is required.

#### Scenario: Client cancels via cancellation link
- **WHEN** a client visits the cancellation URL with a valid token for a confirmed session
- **THEN** the system sets the session status to `cancelled`, deletes the Google Calendar event, and displays a cancellation confirmation

#### Scenario: Client uses an invalid cancellation token
- **WHEN** a client visits a cancellation URL with a token that does not match any session
- **THEN** the system displays an error indicating the cancellation link is invalid

#### Scenario: Client attempts to cancel an already-cancelled session
- **WHEN** a client visits the cancellation URL for a session that is already cancelled
- **THEN** the system displays a message indicating the session has already been cancelled

### Requirement: Session status tracking
The system SHALL track session status as one of: `confirmed`, `cancelled`. Sessions use soft delete — cancelled sessions remain in the database for record-keeping.

#### Scenario: Viewing session status
- **WHEN** a therapist views their session list
- **THEN** each session displays its current status (`confirmed` or `cancelled`)

### Requirement: Therapist views upcoming sessions
The system SHALL provide therapists with a view of their upcoming confirmed sessions, showing the session date, time, and client name.

#### Scenario: Therapist has upcoming sessions
- **WHEN** a therapist views their dashboard
- **THEN** the system displays a list of upcoming confirmed sessions sorted by date

#### Scenario: Therapist has no upcoming sessions
- **WHEN** a therapist views their dashboard and has no confirmed future sessions
- **THEN** the system displays a message indicating there are no upcoming sessions
