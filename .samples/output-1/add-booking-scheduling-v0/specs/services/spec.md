# Capability: Services

Appointment type management for therapists.

## ADDED Requirements

### Requirement: Service Creation

The system SHALL allow therapists to create appointment types (services).

#### Scenario: Create a new service
- **WHEN** a therapist creates a new service
- **THEN** they can specify name, duration, description, and location type
- **AND** the service appears in their service list
- **AND** an audit log entry is created

#### Scenario: Required service fields
- **WHEN** a therapist creates a service
- **THEN** the system requires: name (max 100 chars), duration (in minutes), and location type
- **AND** description is optional (max 500 chars)

#### Scenario: Duration options
- **WHEN** a therapist sets service duration
- **THEN** they can select from predefined options: 30, 45, 60, 90, 120 minutes
- **OR** enter a custom duration between 15 and 180 minutes

### Requirement: Location Type Configuration

The system SHALL support online and in-person appointment types.

#### Scenario: Online appointment configuration
- **WHEN** a therapist configures a service as "online"
- **THEN** bookings for this service will automatically generate a Google Meet link

#### Scenario: In-person appointment configuration
- **WHEN** a therapist configures a service as "in-person"
- **THEN** they can optionally provide a location address
- **AND** bookings for this service will not generate a Meet link

#### Scenario: Both online and in-person
- **WHEN** a therapist configures a service as "both"
- **THEN** clients can choose their preferred format during booking

### Requirement: Service Updates

The system SHALL allow therapists to update existing services.

#### Scenario: Edit service details
- **WHEN** a therapist updates a service
- **THEN** the changes apply to future bookings only
- **AND** existing bookings retain their original service details
- **AND** an audit log entry is created

#### Scenario: Service visibility toggle
- **WHEN** a therapist hides a service
- **THEN** the service no longer appears on the public booking page
- **AND** existing bookings for this service are not affected

### Requirement: Service Deletion

The system SHALL allow therapists to delete services with appropriate safeguards.

#### Scenario: Delete service without bookings
- **WHEN** a therapist deletes a service with no upcoming bookings
- **THEN** the service is permanently removed
- **AND** an audit log entry is created

#### Scenario: Delete service with upcoming bookings
- **WHEN** a therapist attempts to delete a service with upcoming bookings
- **THEN** the system warns them about affected bookings
- **AND** requires confirmation before proceeding
- **AND** offers to hide the service instead of deleting

### Requirement: Service Display Order

The system SHALL allow therapists to control the order services appear on the booking page.

#### Scenario: Reorder services
- **WHEN** a therapist reorders their services
- **THEN** the booking page displays services in the specified order
