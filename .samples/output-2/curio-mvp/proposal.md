## Why

Solo online therapists waste hours each week on manual scheduling — juggling Google Calendar, separate booking tools, and spreadsheets to manage clients and sessions. Existing practice management tools are expensive, complex, and force therapists out of Google Workspace where they already work. We need to validate that a Workspace-native approach works before building the full Curio product: can we deliver reliable Google Calendar sync and an end-to-end booking flow that therapists would actually trust?

## What Changes

- Add Google OAuth login for therapists, establishing Workspace connectivity
- Add a public booking page (`/{slug}/bookings`) where clients book sessions without authentication
- Add two-way Google Calendar sync: bookings create calendar events, existing events block availability
- Add therapist availability settings with time zone configuration
- Add session cancellation flow that syncs back to Google Calendar
- Add basic client management (name, email, phone) with automatic client creation on first booking

## Capabilities

### New Capabilities

- `google-auth`: Google OAuth login for therapists, including Calendar API scope grants and token management
- `booking-page`: Public path-based booking page showing available slots, accepting client info, and creating sessions
- `calendar-sync`: Two-way Google Calendar integration — read existing events for availability, write bookings as events, sync cancellations
- `availability`: Therapist availability configuration (days, hours, time zone) used to compute bookable slots
- `session-management`: Session lifecycle — creation from bookings, cancellation by therapist or client, status tracking
- `client-management`: Basic client records (name, email, phone) with automatic creation on first booking and therapist-facing list view

### Modified Capabilities

_(No existing capabilities — this is the initial MVP)_

## Impact

- **New application**: Full Next.js app with Supabase backend, deployed on Vercel
- **External APIs**: Google Calendar API (read/write), Google OAuth 2.0
- **Database**: New Supabase tables for therapists, clients, sessions, availability
- **Public surface**: Unauthenticated booking pages exposed at `/{slug}/bookings`
- **Dependencies**: Google Cloud project with OAuth consent screen and Calendar API enabled
