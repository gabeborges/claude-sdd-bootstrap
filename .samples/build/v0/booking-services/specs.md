# Booking Services -- Specs

## Goal

Allow therapists to create, edit, and delete appointment types (services) for their practice, establishing the service catalog that clients will book from. This is the foundation of the booking system -- without defined services, no bookings can occur.

Services support both online (with Google Meet) and in-person modalities.

## Non-Goals

- No group session types (v0 is individual appointments only)
- No recurring appointment templates
- No intake forms attached to services
- No buffer time configuration between appointments (nice-to-have, deferred)
- No booking policies (cancellation/rescheduling rules) attached to services
- No pricing or payment information on services (v0 is free)
- No service categories or grouping
- No service-level branding or customization
- No AI-assisted service creation or suggestions
- Not a full EHR feature -- services are appointment types only
- Not for multi-therapist practices (solo therapist only, single tenant)

## Requirements

- R1: Therapists must be able to create a new service/appointment type with the following fields: name, duration (in minutes), modality (online or in-person), and optional description.
- R2: Therapists must be able to edit any existing service they own (name, duration, modality, description).
- R3: Therapists must be able to delete a service. Deletion must not remove or corrupt existing bookings that reference the service.
- R4: Each service must belong to exactly one therapist (tenant isolation). A therapist must not be able to view, edit, or delete another therapist's services.
- R5: Supabase Row Level Security (RLS) must enforce tenant isolation for all service CRUD operations. No application-level-only authorization.
- R6: All service create, update, and delete operations must emit an audit log entry (actor, action, timestamp, affected record, before/after values for updates).
- R7: Service data must be encrypted at rest (Supabase default encryption) and in transit (TLS 1.2+).
- R8: Services with modality "online" must result in Google Meet link generation when a booking is made against them (Meet link generation itself is the responsibility of google-calendar-sync; this spec only requires the modality flag is stored and propagated).
- R9: Services with modality "in-person" must not generate a Meet link. The system must clearly distinguish between online and in-person services.
- R10: The service list displayed on the public booking page must only include active services (requires design gate: determine whether services have an active/inactive status or if deletion is the only removal mechanism).
- R11: Service names must be non-empty and reasonably bounded in length (requires design gate: define max length).
- R12: Duration must be a positive integer in minutes with a reasonable minimum (e.g., 15 minutes) and maximum (requires design gate: define bounds).
- R13: All service data must be stored in Canada-region infrastructure (data residency requirement).
- R14: The UI for service management must be WCAG compliant (keyboard navigable, screen reader accessible, sufficient color contrast).

## Acceptance Criteria (Scenarios)

### S1: Therapist creates a new online service
**Given** a therapist is authenticated and on their service management page
**When** they fill in name ("Initial Consultation"), duration (60 minutes), modality (online), description ("First session for new clients"), and submit
**Then** the service is created, appears in their service list, and an audit log entry is recorded for the create action

### S2: Therapist creates a new in-person service
**Given** a therapist is authenticated and on their service management page
**When** they fill in name ("In-Office Follow-up"), duration (45 minutes), modality (in-person), and submit
**Then** the service is created with modality "in-person" and no Meet link configuration is associated

### S3: Therapist edits an existing service
**Given** a therapist has a service "Follow-up Session - 45 min"
**When** they change the name to "Follow-up Session - 50 min" and duration to 50 minutes and save
**Then** the service is updated, the change is reflected in the service list, and an audit log entry records the before/after values

### S4: Therapist deletes a service with no existing bookings
**Given** a therapist has a service with no bookings
**When** they delete the service
**Then** the service is removed from the service list and an audit log entry is recorded

### S5: Therapist deletes a service that has existing bookings
**Given** a therapist has a service "Initial Consultation" with 3 upcoming bookings
**When** they delete the service
**Then** the service is removed from the booking page (no new bookings can be made), existing bookings remain intact and accessible, and an audit log entry is recorded

### S6: Tenant isolation prevents cross-therapist access
**Given** Therapist A has created service "Therapy Session"
**When** Therapist B attempts to view, edit, or delete Therapist A's service (via API or UI)
**Then** the request is denied by RLS policy and Therapist B receives no data about Therapist A's services

### S7: Service creation with missing required fields
**Given** a therapist is creating a new service
**When** they submit the form with the name field empty
**Then** the form displays a validation error and the service is not created

### S8: Service creation with invalid duration
**Given** a therapist is creating a new service
**When** they enter a duration of 0 or a negative number
**Then** the form displays a validation error and the service is not created

### S9: Service list is accessible
**Given** a therapist is using a screen reader
**When** they navigate the service management page
**Then** all services, actions (create, edit, delete), and form fields are accessible via keyboard navigation and announced by screen reader

## Open Questions

- Q1: Should services have an "active/inactive" toggle, or is deletion the only way to remove a service from the booking page? Active/inactive would allow therapists to temporarily hide a service without losing it. (Requires design gate.)
- Q2: What are the exact minimum and maximum duration bounds for a service? The PRD gives examples of 45 and 60 minutes but does not specify limits. (Requires design gate.)
- Q3: What is the maximum character length for service name and description fields? (Requires design gate.)
- Q4: Should there be a maximum number of services per therapist? (Requires design gate.)
