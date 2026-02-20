# PRD: Curio MVP

**Version:** v0
**Product Vision Reference:** See `.ops/product-vision-strategy.md` for strategic context

---

## What We're Building

Curio v0 is the initial MVP to prove that a Google Workspace-integrated practice management tool works for solo online therapists. The focus is narrow: session scheduling via a public booking page synced with Google Calendar, and basic client management.

This is a technical proof of concept. If the Google Calendar integration is reliable and the booking flow works end-to-end, we've validated the core approach that the rest of Curio will be built on.

---

## Who It's For

- **Primary users:** Solo online therapists (one therapist, fully remote practice)
- **Secondary users:** Clients booking sessions through the public booking page
- **Context:** Therapists set up their availability, clients book through a public page, and everything syncs to the therapist's Google Calendar

---

## The Problem We're Solving

Solo online therapists spend too much time managing scheduling manually. They juggle between calendar apps, booking tools, and spreadsheets to track clients and sessions. Existing practice management tools are expensive, complex, and don't integrate well with Google Workspace — the tools therapists already live in.

Before building the full product, we need to prove that a Workspace-native approach actually works: that Google Calendar sync is reliable, that a simple booking flow can replace standalone scheduling tools, and that therapists would trust this as part of their daily workflow.

---

## Goals & Outcomes (This Version)

**Primary goal:**
- Prove the Google Workspace integration approach works — specifically that Google Calendar sync is reliable and a booking-to-session flow can work end-to-end

**Secondary goal:**
- Establish the technical foundation (auth, calendar integration, data model) that future versions will build on

These goals are validation-focused, not growth-focused.

---

## Must-Have Features (v0)

These are the **non-negotiables** for this version:

1. **Google OAuth login** – Therapists sign in with their Google account. This also establishes the connection to their Google Workspace.

2. **Public booking page** – A path-based public page (`/{slug}/bookings`) where clients can view available time slots and book a session. No authentication required for clients.

3. **Google Calendar two-way sync** – When a client books a session, a calendar event is created on the therapist's Google Calendar. The therapist's existing calendar events are respected when showing availability (no double-booking).

4. **Therapist availability settings** – Therapists can configure their available hours and set their time zone. All booking slots are displayed based on the therapist's time zone.

5. **Session cancellation** – Clients or therapists can cancel a booked session. Cancellation syncs to Google Calendar (event removed or marked cancelled).

6. **Client management** – Basic client records storing name, email, and phone number. Clients are created when they book their first session. Therapists can view and manage their client list.

---

## Nice-to-Have Features (Later)

Valuable features that are **not required for this version**:

- Automated reminders (email/calendar notifications before sessions)
- Rescheduling flow (currently only cancellation is supported)
- Session notes
- Gmail integration for client communication

---

## How We'll Know It's Working

Success signals for this version:

- **Calendar events sync reliably** — no missed bookings, no duplicates, no sync delays that affect usability
- **Booking flow works end-to-end** — a client can visit the public page, see real availability, book a slot, and the event appears on the therapist's Google Calendar
- **No manual intervention needed** — the therapist doesn't need to fix or reconcile calendar entries after bookings

---

## Requirements (More Detail)

### Functional Requirements

- Therapist can sign in via Google OAuth and grant calendar permissions
- Therapist can set a unique slug for their booking page
- Therapist can configure available days/hours and their time zone
- Public booking page shows available slots based on therapist's availability minus existing Google Calendar events
- Client provides name, email, and phone number when booking
- Booking creates a Google Calendar event on the therapist's calendar
- Client record is created automatically on first booking
- Therapist can view a list of all clients with their contact info
- Therapist or client can cancel a session, which updates Google Calendar accordingly
- Booking page is accessible without authentication (public)

---

### Non-Functional Requirements

- Calendar sync should reflect changes within a reasonable timeframe (near real-time for bookings)
- Public booking page should load quickly and work on mobile browsers
- Google Calendar API usage should handle rate limits gracefully

---

## Technical Considerations (High Level)

### Assumptions

- **Frontend:** Next.js, TypeScript, Tailwind CSS, shadcn
- **Backend/Database:** Supabase
- **Deployment:** Vercel
- **Auth:** Google OAuth (for therapist login and Calendar API access)
- **Calendar integration:** Google Calendar API
- **No compliance infrastructure in v0** — this is a technical proof of concept

See `.ops/product-vision-strategy.md` for core platform decisions.

---

### Constraints

- Must use Google OAuth scopes for Calendar read/write access
- Public booking page must work without client authentication
- All times displayed on booking page are in the therapist's configured time zone

---

### Dependencies

- Google Calendar API availability and OAuth consent screen approval
- Supabase for data storage and auth
- Vercel for hosting

---

## Open Questions

- How should the therapist's slug be validated? (uniqueness, allowed characters, length limits)
- Should the booking confirmation be shown in-page only, or also send a confirmation email to the client?
- How should Google Calendar API token refresh be handled when therapists haven't used the app in a while?
- What happens if a therapist's Google Calendar becomes unavailable (API errors)? Graceful degradation vs. blocking bookings?

---

## What's NOT In Scope

- Compliance (HIPAA, PHIPA, SOC2, encryption requirements) — deferred to later versions
- AI features (note drafting, summaries, suggestions)
- Billing and payments (Stripe integration, invoicing)
- Session notes or clinical documentation
- Multi-therapist support (practices with multiple providers)
- Policy engine or audit logging
- Gmail or Google Drive integration
- Rescheduling (only cancellation for v0)
- Automated reminders
- Mobile app

---

## Out-of-Scope Technical Detail

This PRD does **not** define:

- API contracts
- Database schemas
- Detailed system architecture
- Deployment pipelines

Those belong in:
- Technical design docs
- Architecture documents
- Implementation specs

---

*Generated with Clavix Planning Mode*
*Version: v0*
*Generated: 2026-02-07*
