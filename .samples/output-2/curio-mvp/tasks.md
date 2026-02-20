## 1. Project Setup

- [x] 1.1 Initialize Next.js project with TypeScript, Tailwind CSS, and App Router
- [x] 1.2 Install and configure shadcn/ui component library
- [x] 1.3 Set up Supabase project and configure environment variables (SUPABASE_URL, SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY)
- [x] 1.4 Set up Google Cloud project with OAuth consent screen and enable Google Calendar API
- [x] 1.5 Configure environment variables for Google OAuth (GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET)
- [x] 1.6 Create Supabase client utilities (browser client, server client, service role client)

## 2. Database Schema

- [x] 2.1 Create `therapists` table (id, user_id FK to auth.users, name, email, slug UNIQUE, timezone, google_access_token, google_refresh_token, calendar_connected boolean, created_at, updated_at)
- [x] 2.2 Create `availability_rules` table (id, therapist_id FK, day_of_week integer 0-6, start_time time, end_time time, created_at)
- [x] 2.3 Create `clients` table (id, therapist_id FK, name, email, phone, created_at) with unique constraint on (therapist_id, email)
- [x] 2.4 Create `sessions` table (id, therapist_id FK, client_id FK, start_time timestamptz, end_time timestamptz, status text default 'confirmed', google_calendar_event_id text, cancellation_token UUID unique, created_at) with unique constraint on (therapist_id, start_time) where status = 'confirmed'
- [x] 2.5 Enable Row Level Security on all tables and create RLS policies (therapists own their data; service role can insert clients/sessions for public bookings)

## 3. Google OAuth & Auth

- [x] 3.1 Configure Supabase Auth with Google OAuth provider including calendar scopes (calendar.events, calendar.readonly)
- [x] 3.2 Create login page with "Sign in with Google" button
- [x] 3.3 Create OAuth callback route that extracts Google access/refresh tokens from the provider token and upserts the therapist record
- [x] 3.4 Create auth middleware to protect dashboard routes (redirect unauthenticated users to login)
- [x] 3.5 Implement sign-out functionality (destroy Supabase session, redirect to login)
- [x] 3.6 Handle missing calendar scope — detect when calendar permissions are not granted and prompt re-authorization

## 4. Google Calendar Integration

- [x] 4.1 Create Google Calendar API client utility that initializes with stored tokens from the therapist record
- [x] 4.2 Implement token refresh logic — on 401 response, use refresh token to get new access token, update stored tokens, retry once
- [x] 4.3 Implement FreeBusy query function to fetch busy periods for a therapist's primary calendar over a date range
- [x] 4.4 Implement create event function — creates a calendar event with session time, client name, and returns the event ID
- [x] 4.5 Implement delete event function — deletes a calendar event by event ID, handling "already deleted" gracefully
- [x] 4.6 Handle disconnected calendar state — detect invalid/revoked refresh tokens and mark calendar_connected as false on the therapist record

## 5. Therapist Onboarding & Settings

- [x] 5.1 Create onboarding flow after first login — form to set slug, time zone, and session duration
- [x] 5.2 Implement slug validation (lowercase letters, numbers, hyphens only; uniqueness check against database)
- [x] 5.3 Create time zone selector using IANA time zone list
- [x] 5.4 Create settings page where therapist can update slug, time zone, and session duration

## 6. Availability Configuration

- [x] 6.1 Create availability settings UI — day-of-week toggles with start/end time pickers for each day
- [x] 6.2 Support multiple time windows per day (add/remove windows)
- [x] 6.3 Create API route to save availability rules (replace all rules for the therapist on save)
- [x] 6.4 Create API route to fetch availability rules for a therapist

## 7. Slot Generation & Availability Engine

- [x] 7.1 Implement slot generation function — given availability rules, a date range, and session duration, produce candidate time slots in the therapist's time zone
- [x] 7.2 Implement availability engine — combine slot generation with Google Calendar FreeBusy data to return only open slots
- [x] 7.3 Create API route for public slot fetching — accepts therapist slug and date range, returns available slots (uses service role for DB access)

## 8. Public Booking Page

- [x] 8.1 Create dynamic route at `/[slug]/bookings` — server-render therapist name and initial available slots
- [x] 8.2 Build date picker UI for selecting which dates to view slots for
- [x] 8.3 Build time slot selection UI — display available slots, allow client to select one
- [x] 8.4 Build booking form — collect client name, email, phone number with validation
- [x] 8.5 Create booking API route — validate slot availability, find-or-create client, create session with cancellation token, create Google Calendar event, return confirmation
- [x] 8.6 Handle double-booking error — catch unique constraint violation on (therapist_id, start_time), return user-friendly error and refresh slots
- [x] 8.7 Handle Calendar API failure on booking — roll back session creation, return error to client
- [x] 8.8 Build booking confirmation view — display session date/time in therapist's time zone and cancellation link
- [x] 8.9 Handle invalid slug — show 404 page when slug doesn't match any therapist

## 9. Session Management & Cancellation

- [x] 9.1 Create cancellation page at `/cancel/[token]` — look up session by cancellation token, show session details and confirm cancellation button
- [x] 9.2 Create cancellation API route — set session status to cancelled, delete Google Calendar event, handle already-cancelled and invalid token cases
- [x] 9.3 Handle Calendar event deletion failure — still cancel the session in database, log the sync error

## 10. Therapist Dashboard

- [x] 10.1 Create dashboard layout with navigation (dashboard, clients, availability, settings)
- [x] 10.2 Build upcoming sessions view — list confirmed future sessions sorted by date, showing time and client name
- [x] 10.3 Add cancel button to each session with confirmation dialog, triggering cancellation + calendar sync
- [x] 10.4 Show empty state when no upcoming sessions exist
- [x] 10.5 Show calendar connection status — prompt to reconnect if calendar_connected is false

## 11. Client Management

- [x] 11.1 Build client list page — display all clients for the therapist with name, email, phone
- [x] 11.2 Build client detail page — show client contact info and their session history (confirmed and cancelled)
- [x] 11.3 Show empty state when no clients exist
