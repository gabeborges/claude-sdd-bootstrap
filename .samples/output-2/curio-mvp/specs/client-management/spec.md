## ADDED Requirements

### Requirement: Automatic client creation on first booking
The system SHALL create a client record when a new client books a session for the first time. The client record stores name, email, and phone number as provided in the booking form. Each client is linked to the therapist they booked with.

#### Scenario: New client books for the first time
- **WHEN** a client submits a booking with an email address that does not match any existing client for that therapist
- **THEN** the system creates a new client record with the provided name, email, and phone number

#### Scenario: Returning client books again
- **WHEN** a client submits a booking with an email address that matches an existing client for that therapist
- **THEN** the system links the new session to the existing client record without creating a duplicate

### Requirement: Therapist views client list
The system SHALL provide therapists with a list view of all their clients, displaying each client's name, email, and phone number.

#### Scenario: Therapist has clients
- **WHEN** a therapist navigates to the client list
- **THEN** the system displays all clients associated with that therapist, showing name, email, and phone number

#### Scenario: Therapist has no clients
- **WHEN** a therapist navigates to the client list and has no client records
- **THEN** the system displays a message indicating there are no clients yet

### Requirement: Therapist views individual client details
The system SHALL allow therapists to view an individual client's details including their contact information and session history with that therapist.

#### Scenario: Viewing client details
- **WHEN** a therapist selects a client from the client list
- **THEN** the system displays the client's name, email, phone number, and a list of their sessions (confirmed and cancelled)

### Requirement: Client data scoped to therapist
Each client record SHALL be scoped to a single therapist. If the same person books with two different therapists, they SHALL have separate client records. Therapists SHALL only be able to view and manage their own clients.

#### Scenario: Same email books with different therapists
- **WHEN** a client with the same email books sessions with two different therapists
- **THEN** each therapist has their own separate client record for that person

#### Scenario: Therapist attempts to access another therapist's clients
- **WHEN** a therapist attempts to query client records belonging to another therapist
- **THEN** the system returns no results (enforced by Row Level Security)
