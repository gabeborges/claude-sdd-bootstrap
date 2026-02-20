## Context

This is a greenfield MVP — no existing codebase. We're building a Next.js application backed by Supabase that integrates with Google Calendar to provide session scheduling for solo online therapists.

The product vision specifies Next.js + TypeScript + Tailwind CSS + shadcn for frontend, Supabase for backend/database, and Vercel for deployment. Google OAuth handles both authentication and Calendar API authorization in a single flow.

The PRD scopes v0 tightly: public booking page, calendar sync, availability settings, session cancellation, and client management. No compliance, no AI, no billing, no multi-therapist support.

## Goals / Non-Goals

**Goals:**
- Prove Google Calendar two-way sync is reliable (the primary validation goal)
- Deliver an end-to-end booking flow: client visits page → sees availability → books → event appears on therapist's calendar
- Establish a clean data model and project structure that future versions can build on
- Keep the architecture simple — this is a proof of concept

**Non-Goals:**
- Compliance infrastructure (HIPAA, PHIPA, audit logging, encryption at rest)
- AI features of any kind
- Billing, payments, or invoicing
- Multi-therapist or practice-level features
- Real-time collaborative features
- Mobile app or PWA
- Webhook-based calendar sync (polling/on-demand is sufficient for v0)

## Decisions

### 1. Auth: Supabase Auth with Google OAuth provider

**Decision:** Use Supabase Auth's built-in Google OAuth provider for therapist login, and separately manage Google API tokens for Calendar access.

**Why:** Supabase Auth handles session management, JWT tokens, and refresh flows out of the box. For Calendar API access, we need to request additional OAuth scopes (`calendar.events`, `calendar.readonly`) beyond basic profile info. We'll store the Google access/refresh tokens in a `therapists` table after the OAuth callback.

**Alternatives considered:**
- *Custom OAuth flow entirely*: More control but duplicates what Supabase Auth already provides (session management, cookie handling, middleware)
- *Supabase Auth only (no extra scopes)*: Wouldn't give us Calendar API access

### 2. Calendar sync: On-demand reads + synchronous writes

**Decision:** Read the therapist's Google Calendar on-demand when computing available slots. Write events synchronously when a booking is created or cancelled.

**Why:** For v0, on-demand is simpler and avoids the complexity of background sync jobs, webhook registration, and stale cache invalidation. The Google Calendar API free-busy endpoint is fast enough for single-therapist reads.

**Alternatives considered:**
- *Background sync with local cache*: Better performance but adds significant complexity (sync jobs, conflict resolution, staleness). Premature for a proof of concept
- *Google Calendar push notifications (webhooks)*: Requires a publicly accessible endpoint and webhook management. Adds infra complexity. Better suited for v1 when we need real-time sync

**Trade-off:** Slightly slower booking page load (API call per page view) in exchange for zero sync infrastructure.

### 3. Availability computation: Server-side slot calculation

**Decision:** Compute available booking slots on the server by intersecting the therapist's configured availability windows with their Google Calendar free/busy data.

**Why:** Server-side computation keeps the logic in one place, avoids exposing Google Calendar data to the client, and ensures consistent time zone handling. The therapist's configured time zone is the canonical reference for all slot calculations.

**Flow:**
1. Load therapist's availability rules (e.g., Mon-Fri 9am-5pm EST)
2. Generate candidate slots for the requested date range
3. Query Google Calendar FreeBusy API for that range
4. Subtract busy periods from candidate slots
5. Return available slots to the booking page

### 4. Public booking page: Next.js dynamic route

**Decision:** Use a Next.js dynamic route at `/[slug]/bookings` for the public booking page.

**Why:** Path-based routing is a PRD requirement. Dynamic routes in Next.js handle this naturally. The page will be server-rendered for the initial load (fetching therapist info and available slots) with client-side interactions for date selection and booking submission.

**Alternatives considered:**
- *Subdomain-based routing*: More professional URLs but requires wildcard DNS and more complex hosting config. Overkill for v0
- *Static generation (SSG)*: Availability changes frequently, so static pages would be stale immediately

### 5. Database: Flat schema with Supabase tables

**Decision:** Use a straightforward relational schema in Supabase with these core tables:
- `therapists` — profile, slug, time zone, Google tokens
- `availability_rules` — weekly recurring availability windows per therapist
- `clients` — name, email, phone, linked to therapist
- `sessions` — booking details, status, Google Calendar event ID, linked to therapist + client

**Why:** Simple, normalized, easy to query. The Google Calendar event ID stored on sessions enables sync operations (update/delete events). No need for complex schemas at this stage.

**Row Level Security (RLS):** Enable RLS on all tables. Therapists can only access their own data. Public booking page queries use a service role key scoped to read availability and create sessions/clients.

### 6. Session cancellation: Soft delete with calendar sync

**Decision:** Cancelling a session sets its status to `cancelled` (soft delete) and deletes or cancels the corresponding Google Calendar event.

**Why:** Soft delete preserves booking history for the therapist's records. The Google Calendar event is removed so the time slot becomes available again.

**Who can cancel:**
- Therapist: via the authenticated dashboard
- Client: via a cancellation link (with a unique token in the URL). No client authentication required.

### 7. Token storage: Encrypted Google tokens in therapists table

**Decision:** Store Google OAuth access and refresh tokens in the `therapists` table. Use Supabase Vault or a simple encrypted column for the refresh token.

**Why:** We need persistent access to the therapist's Google Calendar, even when they're not actively using the app. The refresh token allows us to obtain new access tokens as needed.

**Token refresh:** Handle token expiry transparently — if a Calendar API call fails with 401, refresh the token and retry once.

## Risks / Trade-offs

**[Google API rate limits]** → The Calendar API has per-user rate limits. For v0 with few users, this is unlikely to be an issue. If it becomes a problem, add short-lived caching of free/busy results (e.g., 1-2 minute TTL).

**[Token expiry / revocation]** → If a therapist revokes Google access or their refresh token expires (6 months of inactivity for unverified apps), Calendar sync breaks silently. → Detect auth failures on Calendar API calls and surface a re-authentication prompt to the therapist.

**[Booking race conditions]** → Two clients could book the same slot simultaneously if the Google Calendar hasn't been updated between requests. → Use a database-level unique constraint on (therapist_id, start_time) for sessions with status `confirmed` to prevent double-booking at the data layer.

**[Unverified OAuth app limitations]** → Google limits unverified apps to 100 users and shows a warning screen. → Acceptable for v0 proof of concept. Submit for verification before broader launch.

**[On-demand Calendar reads are slower than cached]** → Each booking page load hits the Google Calendar API. → Acceptable latency for v0. The FreeBusy API is fast (~200-400ms). Cache if needed later.

**[Public booking page abuse]** → No authentication on the booking page means bots could spam bookings. → Basic mitigation: rate limiting on the booking endpoint. Add CAPTCHA or email verification in v1 if needed.

## Open Questions

- Should we use Supabase Vault for Google token encryption, or is a simpler approach (e.g., pgcrypto) sufficient for v0 given no compliance requirements?
- What's the maximum booking window? (e.g., can clients book 30 days out? 90 days?)
- What session durations should be supported? Fixed (e.g., 50 minutes) or configurable per therapist?
- Should the cancellation link include an expiry, or can clients cancel anytime before the session?
