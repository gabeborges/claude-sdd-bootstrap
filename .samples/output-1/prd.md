# PRD: Booking & Scheduling

**Version:** v0
**Product Vision Reference:** See `ops/product-vision-strategy.md` for strategic context

---

## What We're Building

A booking and scheduling system that allows solo online therapists to manage their availability and services while enabling their clients to self-book appointments without back-and-forth communication.

This is the first vertical feature of Curio's practice management wedge, integrating directly with Google Calendar and establishing the foundational patterns (policy checks, audit logging, Google Workspace integration) that future verticals will build upon.

---

## Who It's For

- **Primary users:** Clients booking therapy appointments
- **Secondary users:** Therapists configuring the booking system (services, availability, practice details)
- **Context:** Clients access a public booking page to schedule appointments. Therapists set up their services and availability once, then manage bookings as needed with minimal ongoing effort.

---

## The Problem We're Solving

Solo online therapists currently use Google Booking or similar generic tools to manage appointments. This approach has significant limitations:

- **Too generic:** Not designed for therapy practices or healthcare workflows
- **Missing features:** No client management, intake forms, or documentation workflow integration
- **No compliance management:** Therapists must figure out HIPAA/PHIPA requirements on their own
- **Workflow friction:** Too many manual steps, tool-switching, and back-and-forth communication with clients

Therapists need a booking solution that understands their practice, reduces admin time, and handles compliance by default.

---

## Goals & Outcomes (This Version)

**Primary goal:**
- Clients can book and manage their appointments without back-and-forth emails or communication

**Secondary goal:**
- Therapists spend minimal time on booking administration

These outcomes demonstrate that Curio can reduce admin burden and fit naturally into therapist workflows.

---

## Must-Have Features (v0)

These are the **non-negotiables** for this version:

1. **Service/appointment CRUD** - Therapists can create, edit, and delete appointment types (e.g., "Initial Consultation - 60 min", "Follow-up Session - 45 min")

2. **Availability management** - Therapists can set their available days and hours, with support for multiple time blocks per day (e.g., 9am-12pm and 2pm-5pm)

3. **Public booking page** - Clients can view available services, select a time slot, and complete booking by providing their details (name, email, phone number, consent)

4. **Booking management** - Both clients and therapists can reschedule or cancel appointments. Therapists can also create bookings manually.

5. **Booking emails & reminders** - Automated emails for booking confirmation, rescheduling, cancellation, and appointment reminders (day before + 1 hour before)

6. **Google Calendar sync** - One-way sync from Curio to Google Calendar with automatic Google Meet link generation for online appointments

7. **Online and in-person support** - Appointments can be configured as online (with Meet link) or in-person

---

## Nice-to-Have Features (Later)

Valuable features that are **not required for this version**:

- **Booking policies** - Configurable cancellation and rescheduling rules
- **Name display format selection** - Privacy control for how client names appear on Google Calendar
- **Buffer time** - Automatic gaps between appointments
- **Customizable reminders** - Therapist-defined reminder timing

---

## How We'll Know It's Working

Success signals for this version:

- **Bookings per week** - Therapists are actively receiving client bookings through Curio
- **Fewer manual interventions** - Reduced back-and-forth communication for scheduling

These tie back to our goals: if booking is working, clients self-serve and therapists spend less time on admin.

---

## Requirements (More Detail)

### Functional Requirements

Required system behaviors:

- **Timezone handling** - Therapists must select their timezone during setup. Client-facing booking pages display availability in the therapist's timezone.

- **Double-booking prevention** - When a client selects a time slot, it must be locked/held until they complete the booking or a timeout period expires. This prevents race conditions.

- **Minimum notice** - Therapists can configure a minimum lead time for bookings (e.g., "no same-day bookings" or "at least 24 hours notice")

- **Client data collection** - Booking requires: email, name, phone number, and consent to store personal information

- **Booking URL format** - Public booking pages accessible at `www.curio.health/{therapist-identifier}/booking`

---

### Non-Functional Requirements

- **Performance** - Booking page must load in ~0.2 seconds (fast and feels fast)
- **Accessibility** - WCAG compliant (required, not optional)
- **Uptime** - 99.9% availability target
- **Reminder timing** - Reminders sent day before and 1 hour before appointment (customizable in future versions)

---

## Technical Considerations (High Level)

*This section respects the Product Technical Strategy from ops/product-vision-strategy.md*

### Assumptions

Engineers should assume the following technical decisions (see `ops/product-vision-strategy.md` for details):

- **Frontend:** Next.js, TypeScript, Tailwind CSS, shadcn, headless UI
- **Backend/Database:** Supabase
- **Deployment:** Vercel
- **Email:** Resend (for booking confirmations and reminders)
- **Google Workspace:** OAuth for authentication, Calendar API for sync, Meet for online appointments

v0 establishes foundational patterns:
- Basic policy check abstraction
- Audit event logging
- Google OAuth + identity foundation
- Data model patterns supporting compliance (tenant isolation, timestamps, actor tracking)

---

### Constraints

- **HIPAA/PHIPA compliance from day 0** - All booking data is PHI-adjacent. Encryption at rest and in transit, audit logging, access controls, and data minimization are required.

- **Data residency: Canada only** - v0 serves Canadian therapists only. All data must be stored in Canada. US data residency will be added when expanding to US market.

- **No silent writes** - Per product vision, Curio never modifies Google Workspace data without explicit user/system action + audit entry.

---

### Dependencies

- **Google OAuth** - Required for therapist authentication and identity
- **Therapist onboarding flow** - Therapists must complete setup (services, availability) before booking is functional
- **v0 is free** - No payment or subscription integration required for this version

---

## Integration Points

**Google Calendar (Primary Integration)**
- One-way sync: Curio creates calendar events when bookings are made/modified
- Google Meet link generation for online appointments
- Curio stores event references, not duplicates (Calendar remains source of truth for the event)

**Google OAuth**
- Therapist authentication and identity
- Minimal OAuth scopes per product vision (least privilege)

---

## Open Questions

Items to resolve during implementation:

- **UX/Design** - Not finalized for v0. Constraint: avoid purple/blue color palette. Design polish is not a v0 priority.
- **Branding customization** - Scope undefined, not a v0 priority
- **Booking URL structure** - Exact format TBD (e.g., `curio.health/{username}/booking` vs `curio.health/book/{id}`)

---

## What's NOT In Scope

Explicit exclusions for v0:

**Features excluded:**
- Payments or deposits at booking time
- Group session booking
- Recurring appointments
- Two-way calendar sync (only one-way: Curio → Google Calendar)
- Client accounts or client login
- Multiple therapist support (practices/teams)
- Intake forms attached to booking
- Booking policies (cancellation/rescheduling rules)
- Buffer time between appointments
- Name display format selection

**Users excluded:**
- Multi-therapist practices (v0 is solo therapists only)
- US-based therapists (Canada only for v0)

**Technical scope excluded:**
- Design polish or branding customization
- Mobile app (web only)
- Offline functionality

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
*Generated: 2025-01-22*
