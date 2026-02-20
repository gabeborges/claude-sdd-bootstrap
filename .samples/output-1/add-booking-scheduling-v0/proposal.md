# Change: Add Booking & Scheduling System (v0)

## Why

Solo online therapists spend significant time on scheduling administration - back-and-forth emails, manual calendar management, and coordinating appointment details. Existing tools like Google Booking are too generic for therapy practices, lacking healthcare-specific workflows and compliance management. Curio's first vertical feature will eliminate this friction by providing a booking system that understands therapy practice workflows and handles compliance by default.

## What Changes

This proposal introduces the complete v0 booking and scheduling system:

- **Therapist Onboarding** - Setup flow for timezone, profile, and Google Workspace connection
- **Services** - CRUD for appointment types (e.g., "Initial Consultation - 60 min") with online/in-person configuration
- **Availability** - Weekly schedule management with multiple time blocks per day
- **Public Booking Page** - Client-facing page for viewing services and booking appointments
- **Booking Management** - Reschedule, cancel, and manual booking capabilities for therapists and clients
- **Booking Notifications** - Automated emails for confirmations, reminders (day before + 1 hour), and cancellations
- **Google Calendar Sync** - One-way sync (Curio → Calendar) with automatic Google Meet link generation
- **Audit Foundation** - Core audit logging patterns for compliance (HIPAA/PHIPA readiness)

## Impact

- **Affected specs:** None (greenfield - creates 8 new capabilities)
- **Affected code:** Entire application (new codebase)
- **External dependencies:**
  - Google OAuth (therapist authentication)
  - Google Calendar API (event creation, Meet links)
  - Resend (transactional emails)
  - Supabase (database, auth)
  - Vercel (deployment)

## Constraints

- **HIPAA/PHIPA compliance from day 0** - All booking data is PHI-adjacent; encryption, audit logging, and access controls required
- **Canada-only data residency** - All data must be stored in Canada for v0
- **No silent writes** - Curio never modifies Google Workspace without explicit action + audit entry
- **Performance** - Booking page must load in ~0.2 seconds
- **Accessibility** - WCAG compliance required

## Out of Scope (v0)

- Payments or deposits
- Group session booking
- Recurring appointments
- Two-way calendar sync
- Client accounts/login
- Multi-therapist practices
- Intake forms
- Booking policies (cancellation rules)
- Buffer time between appointments
- US data residency
