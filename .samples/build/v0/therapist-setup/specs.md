# Therapist Setup -- Specs

## Goal

Provide a guided onboarding flow for therapists to configure their practice and connect their Google account, so they can start receiving bookings. This is the entry point to the entire booking system -- without completing setup, no booking page is functional. The setup flow must be simple, fast, and collect only the minimum information needed to launch booking.

## Non-Goals

- No multi-therapist practice setup (v0 is solo therapists only)
- No team/staff management
- No branding or visual customization of the booking page
- No practice logo upload
- No intake form configuration
- No billing or subscription setup (v0 is free)
- No client import or migration from other tools
- No US-based therapist support (Canada only in v0)
- No non-Google authentication (Google OAuth is the only auth method)
- Not a full EHR onboarding -- this is booking-specific setup only
- No AI-assisted setup or recommendations

## Requirements

- R1: Therapists must authenticate via Google OAuth. Google is the sole identity provider for v0.
- R2: OAuth scopes must follow least-privilege principle. The initial authentication scope should be limited to profile and email. Calendar-specific scopes should be requested when the therapist connects Google Calendar (incremental authorization).
- R3: During setup, the therapist must provide their practice name.
- R4: During setup, the therapist must select their timezone from a standard timezone list (e.g., IANA timezone database). The selected timezone is used for all availability and booking display.
- R5: During setup, the therapist must connect their Google Calendar by granting Curio the necessary OAuth permissions for calendar event creation and Meet link generation.
- R6: The setup flow must guide the therapist through creating at least one service (appointment type) before the booking page can go live. (This is a dependency on booking-services.)
- R7: The setup flow must guide the therapist through setting their availability (at least one day with at least one time block) before the booking page can go live. (This is a dependency on availability.)
- R8: The therapist's booking page URL must be generated upon completion of setup (requires design gate: define how the therapist-identifier in the URL is determined -- username, slug, auto-generated).
- R9: The setup flow must clearly communicate progress (e.g., step indicators) so the therapist knows what remains.
- R10: All setup data (practice name, timezone, OAuth tokens) must be encrypted at rest and in transit (TLS 1.2+).
- R11: All setup data must be stored in Canada-region infrastructure.
- R12: OAuth tokens (access and refresh) must be stored encrypted with appropriate key management. Tokens must never be logged or exposed in client-side code.
- R13: Setup completion must emit an audit log entry (actor: therapist, action: "setup.completed", timestamp, therapist ID).
- R14: Google OAuth connection must emit an audit log entry (actor: therapist, action: "google.oauth.connected", timestamp, scopes granted).
- R15: Supabase RLS must enforce that each therapist can only view and modify their own setup/profile data.
- R16: The setup UI must be WCAG compliant.
- R17: If a therapist abandons setup partway through, they must be able to resume from where they left off (setup state is persisted).
- R18: The setup flow must validate that the Google account used for OAuth is a Google Workspace or personal Google account with Calendar access. If Calendar access is unavailable, the therapist must be informed clearly.

## Acceptance Criteria (Scenarios)

### S1: Therapist authenticates via Google OAuth
**Given** a new therapist visits Curio for the first time
**When** they click "Sign in with Google" and authenticate with their Google account
**Then** they are authenticated, their profile (name, email) is populated from Google, and they are directed to the setup flow

### S2: Therapist enters practice details
**Given** a therapist is in the setup flow after authentication
**When** they enter their practice name ("Smith Therapy") and select timezone ("America/Toronto")
**Then** the practice name and timezone are saved and the setup progresses to the next step

### S3: Therapist connects Google Calendar
**Given** a therapist is at the Google Calendar connection step
**When** they click "Connect Google Calendar" and authorize the additional Calendar scopes
**Then** Curio receives and securely stores the OAuth tokens, and an audit log entry records the connection with scopes granted

### S4: Therapist creates first service during setup
**Given** a therapist has connected Google Calendar and is at the service creation step
**When** they create a service "Initial Consultation" (60 min, online)
**Then** the service is created and the setup progresses to the availability step

### S5: Therapist sets initial availability during setup
**Given** a therapist has created at least one service
**When** they set availability for Monday-Friday 9:00 AM - 5:00 PM
**Then** the availability is saved and setup is marked as complete

### S6: Setup completion generates booking page URL
**Given** a therapist has completed all setup steps (practice details, Google Calendar, service, availability)
**When** setup is marked as complete
**Then** the therapist sees their booking page URL and can share it with clients, and an audit log entry records setup completion

### S7: Incomplete setup prevents booking page from going live
**Given** a therapist has authenticated and entered practice details but has NOT created any services
**When** a client navigates to the therapist's booking page URL
**Then** the booking page shows a "not yet available" or equivalent message (not an error)

### S8: Therapist resumes abandoned setup
**Given** a therapist completed steps 1 and 2 (auth + practice details) but closed the browser
**When** they return and sign in again
**Then** they are taken to step 3 (Google Calendar connection) with their previous progress preserved

### S9: OAuth least-privilege scoping
**Given** a therapist authenticates with Google
**When** the initial OAuth flow completes
**Then** only profile and email scopes are requested; Calendar scopes are not requested until the Calendar connection step

### S10: Google Calendar connection failure
**Given** a therapist attempts to connect Google Calendar
**When** the OAuth flow fails or the therapist denies Calendar permissions
**Then** the therapist sees a clear error message explaining that Calendar access is needed for booking to function, with an option to retry

### S11: Setup data is tenant-isolated
**Given** Therapist A has completed setup
**When** Therapist B attempts to access Therapist A's setup data via API
**Then** the request is denied by RLS

### S12: Setup UI is accessible
**Given** a therapist is using keyboard-only navigation
**When** they go through the entire setup flow
**Then** all form fields, buttons, and navigation elements are reachable and operable via keyboard with clear focus indicators

## Open Questions

- Q1: How is the therapist-identifier for the booking URL determined? Options: therapist-chosen username/slug, auto-generated from practice name, or system-generated UUID. (Requires design gate.)
- Q2: Should the setup flow collect any additional practice details beyond name and timezone (e.g., address for in-person appointments, phone number, credentials/license number)? The PRD does not explicitly list these. (Requires design gate.)
- Q3: What happens if a therapist wants to change their timezone after setup? Is it editable in settings, and what happens to existing bookings? (Requires design gate.)
- Q4: Should the setup flow include a preview of what the booking page will look like before going live? (Requires design gate.)
