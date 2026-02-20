## ADDED Requirements

### Requirement: Therapist login via Google OAuth
The system SHALL authenticate therapists using Google OAuth 2.0 via Supabase Auth. Upon successful authentication, the system SHALL create or update a therapist record and store the Google access and refresh tokens for Calendar API access.

#### Scenario: First-time therapist login
- **WHEN** a therapist clicks "Sign in with Google" and completes the Google OAuth consent flow granting profile and calendar scopes
- **THEN** the system creates a new therapist record with their Google profile info (name, email) and stores the Google access and refresh tokens

#### Scenario: Returning therapist login
- **WHEN** a therapist who already has an account signs in with Google
- **THEN** the system updates their stored Google tokens and establishes a new session

#### Scenario: Therapist denies calendar scope
- **WHEN** a therapist completes Google OAuth but denies the calendar permission scope
- **THEN** the system SHALL inform the therapist that calendar access is required and prompt them to re-authorize with the necessary scopes

### Requirement: OAuth scopes include Calendar access
The system SHALL request the `calendar.events` and `calendar.readonly` OAuth scopes in addition to basic profile scopes during the Google OAuth flow. These scopes are required for reading calendar availability and creating/deleting events.

#### Scenario: OAuth consent includes calendar scopes
- **WHEN** the therapist is redirected to Google's OAuth consent screen
- **THEN** the consent screen requests permission to view and manage calendar events

### Requirement: Google token persistence and refresh
The system SHALL store Google OAuth refresh tokens in the therapists table. When an access token expires, the system SHALL use the refresh token to obtain a new access token transparently.

#### Scenario: Access token has expired
- **WHEN** a Calendar API call fails with a 401 unauthorized error
- **THEN** the system uses the stored refresh token to obtain a new access token and retries the API call once

#### Scenario: Refresh token is invalid or revoked
- **WHEN** the system attempts to refresh a token and Google returns an error indicating the refresh token is invalid
- **THEN** the system marks the therapist's calendar connection as disconnected and displays a prompt to re-authenticate with Google

### Requirement: Therapist session management
The system SHALL maintain authenticated sessions for therapists using Supabase Auth. Protected routes SHALL require a valid session.

#### Scenario: Accessing a protected route without a session
- **WHEN** an unauthenticated user attempts to access a protected route (e.g., dashboard, settings)
- **THEN** the system redirects them to the login page

#### Scenario: Therapist logs out
- **WHEN** a therapist clicks "Sign out"
- **THEN** the system destroys the Supabase session and redirects to the login page
