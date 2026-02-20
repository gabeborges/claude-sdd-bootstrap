# Capability: Audit Foundation

Foundational audit logging for HIPAA/PHIPA compliance.

## ADDED Requirements

### Requirement: Audit Log Structure

The system SHALL maintain an append-only audit log for all significant operations.

#### Scenario: Audit entry format
- **WHEN** any auditable action occurs
- **THEN** an audit log entry is created with:
  - Timestamp (UTC)
  - Actor type (therapist, client, system)
  - Actor ID
  - Action type
  - Resource type
  - Resource ID
  - Metadata (action-specific details)
  - Request ID (for tracing)

#### Scenario: Immutable log
- **WHEN** an audit entry is created
- **THEN** it cannot be modified or deleted
- **AND** forms a tamper-evident record

### Requirement: Booking Audit Events

The system SHALL log all booking-related operations.

#### Scenario: Booking created
- **WHEN** a booking is created
- **THEN** an audit entry logs: actor, booking ID, client ID, service ID, scheduled time

#### Scenario: Booking viewed
- **WHEN** a therapist views booking details (including client contact info)
- **THEN** an audit entry logs the access with actor and booking ID

#### Scenario: Booking updated
- **WHEN** a booking is rescheduled or modified
- **THEN** an audit entry logs: actor, booking ID, changes made, reason (if provided)

#### Scenario: Booking cancelled
- **WHEN** a booking is cancelled
- **THEN** an audit entry logs: actor, booking ID, cancellation reason

### Requirement: Client Data Audit Events

The system SHALL log access to client personal information.

#### Scenario: Client record created
- **WHEN** a new client record is created
- **THEN** an audit entry logs: actor, client ID, source (booking, manual entry)

#### Scenario: Client data accessed
- **WHEN** client contact information is displayed
- **THEN** an audit entry logs the access with actor and client ID

#### Scenario: Client data exported
- **WHEN** client information is included in emails or external systems
- **THEN** an audit entry logs: actor, client ID, destination, data fields

### Requirement: Authentication Audit Events

The system SHALL log all authentication events.

#### Scenario: Successful login
- **WHEN** a therapist successfully authenticates
- **THEN** an audit entry logs: therapist ID, timestamp, OAuth provider

#### Scenario: Failed login
- **WHEN** an authentication attempt fails
- **THEN** an audit entry logs: attempted identity, failure reason, timestamp

#### Scenario: Token refresh
- **WHEN** OAuth tokens are refreshed
- **THEN** an audit entry logs the refresh (without exposing tokens)

#### Scenario: Permission grant/revoke
- **WHEN** OAuth scopes are granted or revoked
- **THEN** an audit entry logs the permission change

### Requirement: Configuration Audit Events

The system SHALL log changes to therapist configuration.

#### Scenario: Service modified
- **WHEN** a therapist creates, updates, or deletes a service
- **THEN** an audit entry logs the change with before/after state

#### Scenario: Availability modified
- **WHEN** a therapist modifies their availability schedule
- **THEN** an audit entry logs the change

#### Scenario: Settings modified
- **WHEN** any therapist settings are changed
- **THEN** an audit entry logs the change with before/after values

### Requirement: Audit Log Retention

The system SHALL retain audit logs according to compliance requirements.

#### Scenario: Retention period
- **WHEN** audit logs are stored
- **THEN** they are retained for a minimum of 6 years (HIPAA requirement)

#### Scenario: No automatic deletion
- **WHEN** the retention period is reached
- **THEN** logs are not automatically deleted without explicit review

### Requirement: Audit Log Access

The system SHALL provide controlled access to audit logs.

#### Scenario: Therapist audit access
- **WHEN** a therapist requests their audit logs
- **THEN** they can view logs related to their practice only
- **AND** the access itself is logged

#### Scenario: Audit export
- **WHEN** audit logs need to be exported (e.g., for legal/compliance review)
- **THEN** the export operation is itself logged
- **AND** includes: requestor, date range, export format

### Requirement: Tenant Isolation

The system SHALL enforce strict data isolation between therapist accounts.

#### Scenario: Query isolation
- **WHEN** any database query is executed
- **THEN** row-level security ensures therapists only access their own data

#### Scenario: Cross-tenant access prevention
- **WHEN** an attempt is made to access another therapist's data
- **THEN** the request is denied
- **AND** the attempt is logged as a security event

#### Scenario: Client data isolation
- **WHEN** a client books with multiple therapists
- **THEN** each therapist only sees their own bookings with that client
- **AND** cannot access the client's history with other therapists
