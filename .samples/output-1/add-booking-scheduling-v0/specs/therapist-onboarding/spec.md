# Capability: Therapist Onboarding

Therapist setup and configuration flow for the booking system.

## ADDED Requirements

### Requirement: Google Authentication

The system SHALL authenticate therapists using Google OAuth with minimal required scopes.

#### Scenario: Successful Google sign-in
- **WHEN** a therapist initiates sign-in with Google
- **THEN** the system requests only necessary OAuth scopes (identity, calendar.events)
- **AND** creates or updates the therapist record with Google identity

#### Scenario: OAuth scope consent
- **WHEN** a therapist grants calendar permissions
- **THEN** the system stores the OAuth tokens securely
- **AND** enables Google Calendar sync functionality

#### Scenario: Denied permissions
- **WHEN** a therapist denies required OAuth scopes
- **THEN** the system informs them which features require the denied permissions
- **AND** allows them to retry the authorization

### Requirement: Timezone Configuration

The system SHALL require therapists to select their timezone during initial setup.

#### Scenario: Timezone selection during onboarding
- **WHEN** a therapist completes authentication for the first time
- **THEN** the system prompts them to select their timezone
- **AND** stores the timezone preference for all scheduling operations

#### Scenario: Timezone displayed on booking page
- **WHEN** a client views the booking page
- **THEN** all available times are displayed in the therapist's configured timezone

#### Scenario: Timezone update
- **WHEN** a therapist updates their timezone in settings
- **THEN** the system updates all future availability displays
- **AND** existing bookings retain their original UTC times

### Requirement: Practice Profile

The system SHALL allow therapists to configure basic practice information.

#### Scenario: Profile setup
- **WHEN** a therapist completes the onboarding flow
- **THEN** they can enter their practice name and professional title
- **AND** this information appears on their public booking page

#### Scenario: Booking URL assignment
- **WHEN** a therapist completes onboarding
- **THEN** the system assigns them a unique booking URL based on their username
- **AND** the URL follows the format: curio.health/{username}/booking

### Requirement: Onboarding Completion Gate

The system SHALL prevent booking page activation until minimum setup is complete.

#### Scenario: Incomplete setup
- **WHEN** a therapist has not configured timezone, at least one service, and availability
- **THEN** the booking page displays a "coming soon" message
- **AND** the therapist sees a setup checklist in their dashboard

#### Scenario: Setup complete
- **WHEN** a therapist has configured timezone, at least one service, and availability
- **THEN** the booking page becomes active and accepts client bookings
