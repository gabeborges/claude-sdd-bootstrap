# Compliance Requirements: Curio v0 Booking & Scheduling

## Overview

This document defines the compliance requirements for the Curio v0 booking and scheduling system. v0 serves solo Canadian therapists and their clients. All data handling is governed by HIPAA, PHIPA (Ontario), and PIPEDA (federal Canada). Compliance is non-negotiable from day 0 per the product vision and PRD constraints.

**Applicable Regulations:**
- [x] HIPAA (Health Insurance Portability and Accountability Act) -- US federal, applied proactively for cross-border readiness
- [x] PHIPA (Personal Health Information Protection Act) -- Ontario, Canada
- [x] PIPEDA (Personal Information Protection and Electronic Documents Act) -- Federal Canada
- [x] SOC 2 Type II (baseline security posture)
- [ ] GDPR -- Not applicable in v0 (Canada-only market)

**Regulatory References:**
- HIPAA: 45 CFR 160.103, 45 CFR 164.312
- PHIPA: O. Reg. 329/04
- PIPEDA: SC 2000, c. 5

---

## 1. Data Classification

### 1.1 Classification Key

| Level | Definition | Handling |
|---|---|---|
| **PHI** | Individually identifiable health information. In this context: any data that reveals a client is receiving therapy services from a specific therapist. | Encrypt at rest + in transit (TLS 1.2+), audit all access, never log content, minimum necessary access, RLS enforced |
| **PHI-adjacent** | Data that, in combination with other data, could identify health service usage (e.g., a booking at a therapy practice implies mental health treatment). | Same protections as PHI -- treat as PHI for all technical controls |
| **PII** | Personally identifiable information not linked to health context | Encrypt at rest + in transit, consent required, right to access/correct/delete |
| **Sensitive-internal** | Internal secrets and credentials | Encrypt at rest, never log, never expose client-side, rotate regularly |
| **Public** | Non-identifiable information safe to expose | Standard handling, no special controls |

### 1.2 Entity-Level Classification

#### `therapists` table

| Column | Classification | Handling Requirements |
|---|---|---|
| `id` | Internal identifier | No special handling; used as FK reference |
| `google_id` | PII | Encrypt at rest; never expose in client responses or logs |
| `email` | PII | Encrypt at rest; hash before writing to audit logs; never expose to other therapists |
| `full_name` | PII | Encrypt at rest; display only to the therapist themselves and on their public booking page (therapist-controlled) |
| `practice_name` | Public | Displayed on booking page; no special handling |
| `timezone` | Public | Displayed on booking page; no special handling |
| `slug` | Public | Used in booking URL; no special handling |
| `setup_completed_at` | Internal metadata | No special handling |
| `setup_step` | Internal metadata | No special handling |
| `google_oauth_access_token` | Sensitive-internal | **Field-level encryption required** (Supabase Vault or pgsodium); never log; never expose client-side; rotate on refresh |
| `google_oauth_refresh_token` | Sensitive-internal | **Field-level encryption required**; never log; never expose client-side; treat as long-lived secret |
| `google_oauth_token_expires_at` | Internal metadata | No special handling |
| `google_oauth_scopes` | Internal metadata | May be logged in audit entries (non-sensitive) |
| `google_calendar_connected` | Internal metadata | No special handling |
| `minimum_notice_hours` | Public (configuration) | No special handling |
| `created_at` / `updated_at` | Internal metadata | No special handling |

#### `services` table

| Column | Classification | Handling Requirements |
|---|---|---|
| `id` | Internal identifier | No special handling |
| `therapist_id` | Internal FK | No special handling |
| `name` | Public | Displayed on public booking page |
| `description` | Public | Displayed on public booking page |
| `duration_minutes` | Public | Displayed on public booking page |
| `modality` | Public | Displayed on public booking page |
| `is_active` | Internal metadata | No special handling |
| `created_at` / `updated_at` | Internal metadata | No special handling |

**Note:** Service names and descriptions are public because the therapist deliberately publishes them on the booking page. However, the *existence* of a service at a therapy practice is PHI-adjacent when combined with a client booking. The service data itself is not PHI.

#### `availability` table

| Column | Classification | Handling Requirements |
|---|---|---|
| `id` | Internal identifier | No special handling |
| `therapist_id` | Internal FK | No special handling |
| `day_of_week` | Public | Displayed on booking page (slot computation) |
| `start_time` / `end_time` | Public | Displayed on booking page |
| `created_at` / `updated_at` | Internal metadata | No special handling |

**Note:** Availability data is non-PHI. It describes the therapist's schedule, not client information. It is intentionally public for booking page slot computation.

#### `bookings` table -- CRITICAL PHI-ADJACENT

| Column | Classification | Handling Requirements |
|---|---|---|
| `id` | Internal identifier | No special handling; do not expose to clients in URLs (use `manage_token` instead) |
| `therapist_id` | Internal FK | No special handling |
| `service_id` | PHI-adjacent | A booking for a therapy service reveals health service usage; encrypt at rest |
| `client_name` | **PHI-adjacent** | Encrypt at rest; never log in plaintext; mask in any debug output; display only to therapist and the client who booked |
| `client_email` | **PHI-adjacent** | Encrypt at rest; **hash** (SHA-256) before writing to audit logs or notification_log; never log in plaintext |
| `client_phone` | **PHI-adjacent** | Encrypt at rest; **never log**; never include in emails; never include in calendar events; never expose in API responses except to the therapist |
| `client_consent` | PHI-adjacent (consent record) | Encrypt at rest; retain as proof of consent per PIPEDA |
| `start_time` / `end_time` | PHI-adjacent | A timestamped booking at a therapy practice is PHI-adjacent; encrypt at rest |
| `status` | PHI-adjacent | Encrypt at rest |
| `modality` | PHI-adjacent | Encrypt at rest |
| `meet_link` | PHI-adjacent | Contains link to therapy session; encrypt at rest; include in emails to participants only |
| `manage_token` | Sensitive-internal | **Cryptographically random** (min 256-bit entropy); never log; treat as bearer credential; consider expiry |
| `manage_token_expires_at` | Internal metadata | No special handling |
| `cancellation_reason` | PHI-adjacent | Encrypt at rest; never log content |
| `created_by` | Internal metadata | No special handling; logged in audit trail |
| `created_at` / `updated_at` | Internal metadata | No special handling |

#### `booking_audit_log` table

| Column | Classification | Handling Requirements |
|---|---|---|
| `id` | Internal identifier | No special handling |
| `entity_type` | Internal metadata | No special handling |
| `entity_id` | Internal reference | No special handling (references entity IDs, not PHI content) |
| `action` | Internal metadata | No special handling |
| `actor` | PII-derived | For clients, store as `client:{sha256(email)}` -- never plaintext email |
| `before_state` / `after_state` | **MUST NOT contain PHI/PII** | Store only non-sensitive state (status changes, timestamps, IDs). **Never store client_name, client_email, client_phone, cancellation_reason in these JSONB fields** |
| `metadata` | Internal | Must not contain PHI/PII; may contain scopes, error codes, sync status |
| `created_at` | Internal metadata | No special handling |

#### `slot_locks` table

| Column | Classification | Handling Requirements |
|---|---|---|
| `id` | Internal identifier | No special handling |
| `therapist_id` | Internal FK | No special handling |
| `start_time` / `end_time` | Transient metadata | No special handling (slot locks are ephemeral) |
| `locked_by` | PII-derived | Store as session ID or hashed identifier; **never store client email or name** |
| `expires_at` | Internal metadata | No special handling |
| `created_at` | Internal metadata | No special handling |

#### `google_calendar_events` table

| Column | Classification | Handling Requirements |
|---|---|---|
| `id` | Internal identifier | No special handling |
| `booking_id` | Internal FK | No special handling |
| `therapist_id` | Internal FK | No special handling |
| `google_event_id` | External reference | No special handling |
| `google_calendar_id` | External reference | No special handling |
| `meet_link` | PHI-adjacent | Encrypt at rest; link to therapy session |
| `sync_status` | Internal metadata | No special handling |
| `last_sync_error` | Internal metadata | Must not contain PHI/PII; error messages only |
| `last_synced_at` | Internal metadata | No special handling |
| `created_at` / `updated_at` | Internal metadata | No special handling |

#### `notification_log` table

| Column | Classification | Handling Requirements |
|---|---|---|
| `id` | Internal identifier | No special handling |
| `booking_id` | Internal FK | No special handling |
| `therapist_id` | Internal FK | No special handling |
| `notification_type` | Internal metadata | No special handling |
| `recipient_type` | Internal metadata | No special handling |
| `recipient_email_hash` | PII-derived | Store SHA-256 hash of email, **never plaintext** |
| `status` | Internal metadata | No special handling |
| `resend_message_id` | External reference | No special handling |
| `error_message` | Internal metadata | Must not contain PHI/PII |
| `retry_count` | Internal metadata | No special handling |
| `scheduled_for` / `sent_at` | Internal metadata | No special handling |
| `created_at` | Internal metadata | No special handling |

---

## 2. HIPAA/PHIPA Technical Safeguards

### 2.1 Access Controls (HIPAA 164.312(a)(1), PHIPA s.12)

#### Requirements
- Every user must have a unique identifier (Google OAuth provides this via `google_id`)
- Session timeout must be enforced for therapist authenticated sessions (automatic logoff after inactivity)
- All PHI-adjacent data access must be through authenticated and authorized channels
- RLS must be enforced on every table -- no application-level-only authorization

#### Technical Controls
- **Unique user identification**: Google OAuth `google_id` mapped to `therapists.id` (UUID)
- **Automatic logoff**: Supabase session timeout must be configured; recommended 30 minutes of inactivity for therapist dashboard sessions
- **RLS enforcement**: Every table has RLS enabled with policies per `db-migration-plan.yaml` Phase 9
- **Client access**: Via `manage_token` only, scoped to single booking; no client accounts or persistent sessions
- **Anonymous access**: Limited to public booking page reads (active services, availability) and booking creation; no PHI exposed on anonymous endpoints
- **Server-side data access**: Booking creation and management operations should route through Server Actions or Edge Functions using `service_role` for defense-in-depth, not direct client-to-Supabase

#### Acceptance Checks
- [ ] Every table has RLS enabled with policies matching db-migration-plan.yaml
- [ ] Therapist can only access their own data (verified by attempting cross-tenant access)
- [ ] Client manage_token grants access to only the specific referenced booking
- [ ] Session timeout is configured (30 minutes recommended) and enforced
- [ ] Anonymous users cannot access any PHI-adjacent data except through the defined booking creation flow
- [ ] Booking creation routes through server-side boundary (Server Action or Edge Function), not direct client-to-Supabase insert

### 2.2 Audit Controls (HIPAA 164.312(b), PHIPA s.10(3))

#### Requirements
- Immutable audit log for all PHI-adjacent data access and modifications
- Audit logs must be tamper-evident (INSERT only, no UPDATE or DELETE)
- Log retention: minimum 6 years (HIPAA) / as required by applicable provincial law
- Log content must include: actor, timestamp, action, resource, outcome
- Log content must NOT include: PHI/PII content (names, emails, phone numbers, cancellation reasons)

#### Technical Controls
- `booking_audit_log` table with database-level immutability triggers (prevent UPDATE/DELETE)
- Actor format: therapist UUID for therapist actions, `client:{sha256(email)}` for client actions, `system` for automated actions
- `before_state` and `after_state` JSONB fields must be **sanitized** before insertion: strip all PHI fields (client_name, client_email, client_phone, cancellation_reason); retain only status, timestamps, IDs, modality
- Audit log RLS: therapist can read own audit logs; insert is server-side only in practice

#### Required Audit Events

| Event | Actor | Entity Type | Trigger |
|---|---|---|---|
| `therapist.created` | system | therapist | Google OAuth first login |
| `therapist.updated` | therapist UUID | therapist | Profile/settings change |
| `setup.completed` | therapist UUID | therapist | Setup flow completion |
| `google.oauth.connected` | therapist UUID | therapist | Calendar OAuth grant |
| `google.oauth.revoked` | system | therapist | Token revocation detected |
| `service.created` | therapist UUID | service | Service CRUD |
| `service.updated` | therapist UUID | service | Service CRUD |
| `service.deleted` | therapist UUID | service | Service CRUD |
| `availability.created` | therapist UUID | availability | Availability CRUD |
| `availability.updated` | therapist UUID | availability | Availability CRUD |
| `availability.deleted` | therapist UUID | availability | Availability CRUD |
| `booking.created` | client:{hash} or therapist UUID | booking | Booking creation |
| `booking.rescheduled` | client:{hash} or therapist UUID | booking | Reschedule |
| `booking.cancelled` | client:{hash} or therapist UUID | booking | Cancellation |
| `booking.completed` | system | booking | Status transition |
| `gcal.event.created` | system | gcal_event | Calendar sync |
| `gcal.event.updated` | system | gcal_event | Calendar sync |
| `gcal.event.deleted` | system | gcal_event | Calendar sync |
| `gcal.sync.failed` | system | gcal_event | Sync failure |
| `notification.sent` | system | booking | Email dispatched |
| `notification.failed` | system | booking | Email delivery failure |

#### Acceptance Checks
- [ ] booking_audit_log table blocks UPDATE and DELETE operations (trigger-enforced immutability verified)
- [ ] Every state-changing operation on PHI-adjacent data creates an audit log entry
- [ ] Audit log before_state/after_state JSONB fields never contain client_name, client_email, client_phone, or cancellation_reason
- [ ] Audit log actor field uses hashed email for client actions (never plaintext)
- [ ] All 20+ audit events listed above are implemented and emit correctly

### 2.3 Encryption Requirements (HIPAA 164.312(a)(2)(iv), 164.312(e)(1))

#### Requirements
- All PHI-adjacent data encrypted at rest (AES-256 or equivalent)
- All data encrypted in transit (TLS 1.2+ minimum; TLS 1.3 preferred)
- OAuth tokens require field-level encryption (separate from database-level TDE)
- Encryption keys managed via KMS with rotation capability

#### Technical Controls
- **At rest**: Supabase provides Transparent Data Encryption (TDE) for the entire database; this covers all PHI-adjacent columns
- **Field-level encryption**: `google_oauth_access_token` and `google_oauth_refresh_token` must use Supabase Vault (pgsodium) or application-level encryption with a separate key, not just TDE
- **In transit**: TLS 1.2+ enforced on all endpoints (Supabase default + Vercel default); HSTS headers recommended
- **Key management**: Supabase-managed encryption keys for TDE; Vault-managed keys for field-level encryption; keys must not be stored in application code or environment variables accessible to client-side code

#### Acceptance Checks
- [ ] Supabase project has TDE enabled (encryption at rest for all data)
- [ ] google_oauth_access_token and google_oauth_refresh_token use field-level encryption (Supabase Vault or pgsodium), not just TDE
- [ ] All API endpoints serve over TLS 1.2+ (no unencrypted HTTP)
- [ ] HSTS header is present on all responses
- [ ] Encryption keys are not present in application source code, client-side bundles, or application logs
- [ ] OAuth tokens are never logged in any application log, error output, or debug trace

### 2.4 Transmission Security (HIPAA 164.312(e)(1))

#### Requirements
- All data transmission containing PHI-adjacent information must be encrypted
- Integrity controls must verify data has not been altered during transmission

#### Technical Controls
- TLS 1.2+ on all connections: browser-to-Vercel, Vercel-to-Supabase, Vercel-to-Resend, Vercel-to-Google APIs
- Certificate validation enforced on all outbound connections
- No PHI-adjacent data transmitted over non-TLS channels

#### Acceptance Checks
- [ ] All external API calls (Supabase, Resend, Google Calendar API) use HTTPS
- [ ] No HTTP fallback is configured on any endpoint
- [ ] Browser-to-server communication uses TLS (Vercel enforces this by default)

### 2.5 Data Integrity (HIPAA 164.312(c)(1))

#### Requirements
- PHI-adjacent data must not be altered or destroyed improperly
- Booking state transitions must follow defined state machine

#### Technical Controls
- Booking status CHECK constraint: `('confirmed', 'cancelled', 'completed')`
- Audit log immutability (no UPDATE/DELETE)
- Database constraints enforce data validity (NOT NULL, CHECK, FK)
- `updated_at` triggers provide modification timestamps

#### Acceptance Checks
- [ ] Booking status transitions follow defined state machine (confirmed -> cancelled, confirmed -> completed)
- [ ] Database CHECK constraints prevent invalid data states
- [ ] All booking modifications are recorded in audit log with before/after state

### 2.6 Authentication (HIPAA 164.312(d), PHIPA s.12(1))

#### Requirements
- All users must be authenticated before accessing PHI-adjacent data
- Authentication must use a verified identity provider

#### Technical Controls
- Google OAuth is the sole identity provider
- Supabase Auth handles session management
- `auth.uid()` used in all RLS policies for therapist identification
- Client access is token-based (manage_token) -- no authentication required, but access is scoped to single booking

#### Acceptance Checks
- [ ] No PHI-adjacent data is accessible without authentication (therapist) or valid manage_token (client)
- [ ] auth.uid() is used in all therapist-facing RLS policies
- [ ] manage_token is cryptographically random with sufficient entropy (minimum 256 bits)

---

## 3. PIPEDA Requirements (Federal Canada)

### 3.1 Consent Management (PIPEDA Principle 3)

#### Requirements
- Explicit, informed consent must be obtained before collecting personal information
- Purpose of collection must be clearly communicated
- Consent must be an affirmative action (opt-in, not opt-out)
- Consent record must be retained as proof

#### Technical Controls
- `bookings.client_consent` field: boolean, CHECK constraint requires `true` for booking creation
- Consent checkbox on booking page must be unchecked by default (explicit opt-in)
- Consent language must clearly state: (a) what data is collected (name, email, phone), (b) purpose (scheduling therapy appointment), (c) who stores the data (Curio on behalf of the therapist), (d) how long it is retained
- `client_consent = true` stored with every booking as proof of consent

#### Acceptance Checks
- [ ] Booking form consent checkbox is unchecked by default
- [ ] Consent language clearly states what data is collected and for what purpose
- [ ] Booking creation is rejected if client_consent is not true (DB CHECK constraint + application validation)
- [ ] Consent record (client_consent = true + created_at timestamp) is retained with the booking record

### 3.2 Purpose Limitation (PIPEDA Principle 4)

#### Requirements
- Personal information must be used only for the purpose for which it was collected
- Client data collected for booking must not be used for marketing, analytics, or other purposes

#### Technical Controls
- Client data (name, email, phone) used exclusively for: (a) booking management, (b) booking lifecycle emails, (c) therapist practice management
- No analytics tracking of client PII
- No sharing of client data with third parties beyond what is necessary for the booking service (Resend for email, Google Calendar for event creation)
- Email addresses sent to Resend for transactional email only; not added to any mailing list

#### Acceptance Checks
- [ ] Client data is not used in any analytics or tracking system
- [ ] Client email is only sent to Resend for booking lifecycle emails (never for marketing)
- [ ] Client data is not shared with any third party beyond Resend (email) and Google Calendar (event creation)

### 3.3 Data Retention (PIPEDA Principle 5)

#### Requirements
- Personal information must be retained only as long as necessary for the identified purpose
- Retention periods must be defined and documented
- Secure deletion must be available when retention period expires

#### Defined Retention Periods

| Data Type | Retention Period | Rationale |
|---|---|---|
| Active bookings | Indefinite while booking is upcoming/active | Required for service delivery |
| Completed bookings | 10 years after completion | PHIPA s.13 requires health records retention; provincial requirements vary (Ontario: 10 years after last contact for adults) |
| Cancelled bookings | 10 years after cancellation | Same as completed bookings (cancellation is part of health record) |
| Audit logs | 10 years | Must equal or exceed data retention period; HIPAA minimum is 6 years |
| Notification logs | 2 years | Operational record, not health record |
| OAuth tokens | Until revoked or therapist account deleted | Required for service operation |
| Slot locks | Ephemeral (auto-expire within minutes) | Transient operational data; clean up expired locks |

#### Technical Controls
- v0 does not implement automated retention/deletion (acceptable for launch given no data exists yet)
- Retention policy must be documented and a backlog item created for automated implementation before production data accumulates
- When deletion is implemented: secure deletion from primary database, replicas, backups (cryptographic erasure via key rotation)

#### Acceptance Checks
- [ ] Retention periods are documented (this document serves as the record)
- [ ] Booking records are never hard-deleted in v0 (status change to cancelled, not row deletion)
- [ ] Expired slot_locks are cleaned up (either filtered at query time or deleted by scheduled job)
- [ ] A backlog item exists for implementing automated retention/deletion before production scale

### 3.4 Right to Access (PIPEDA Principle 9)

#### Requirements
- Individuals must be able to access their personal information held by the organization
- Access must be provided within a reasonable timeframe (30 days under PIPEDA)

#### Technical Controls for v0
- Clients can view their booking details via manage_token link (name, email, service, date/time)
- Therapist can view all their own data via authenticated dashboard
- Formal Subject Access Request (SAR) process is not automated in v0; manual process via therapist is acceptable for launch

#### Acceptance Checks
- [ ] Client can view their booking details via manage_token
- [ ] Therapist can view all their own profile, services, availability, and booking data

### 3.5 Right to Correction (PIPEDA Principle 9)

#### Requirements
- Individuals must be able to challenge the accuracy of their personal information and request corrections

#### Technical Controls for v0
- Clients can update their booking (reschedule/cancel) via manage_token
- Client cannot self-correct their name, email, or phone on an existing booking in v0 (this must go through the therapist)
- Therapist can update booking details on behalf of clients

#### Acceptance Checks
- [ ] Therapist can update client details on bookings they own
- [ ] A process exists (even if manual) for clients to request corrections to their personal information

---

## 4. Data Residency

### 4.1 Canada-Only Requirement (v0)

#### Requirements
- All data must be stored in Canada (per PRD constraint)
- No PHI-adjacent data may leave Canada except where technically necessary for third-party service operation (with appropriate safeguards)

#### Infrastructure Requirements

| Component | Residency Requirement | Verification Method |
|---|---|---|
| Supabase (database, auth, edge functions) | **Canada region** (ca-central-1 or equivalent) | Verify Supabase project region in dashboard settings |
| Vercel (Next.js hosting) | **Canada or acceptable region** -- Vercel edge functions should be configured to prefer Canadian PoPs | Verify Vercel deployment region configuration |
| Resend (email delivery) | Email data transits through Resend infrastructure | **BAA/DPA required with Resend**; verify data processing location; email content should be minimized (data minimization in templates) |
| Google Calendar API | API calls transit through Google infrastructure; event data stored on Google servers (US) | **This is a data residency concern** -- see section 4.2 |

### 4.2 Google Calendar Data Flow -- Data Residency Analysis

**Concern:** When Curio creates a Google Calendar event, the event data (which includes client name and appointment time -- PHI-adjacent) is stored on Google's servers, which are primarily US-based.

**Assessment:**
- The therapist explicitly connects their Google Calendar and consents to this data flow
- Google Calendar is the therapist's own tool; Curio is creating events on the therapist's behalf
- The event data is limited to: service name, client name, appointment time (data minimization applies to event title/description format)
- Google Workspace may store data outside Canada depending on the therapist's Google account configuration

**Required Controls:**
- Data minimization in calendar event content (limit PHI in event title/description -- design gate Q1 in google-calendar-sync specs)
- Audit logging of every calendar write
- Therapist consent for calendar integration (obtained during setup)
- Google BAA/DPA coverage (Google Workspace already provides BAA for healthcare customers)

**Residency Determination:**
- This data flow is **acceptable** because: (a) the therapist consents to it, (b) data is minimized, (c) Google provides BAA coverage, (d) the therapist's Google account is their own tool
- **However**, the calendar event title/description format must minimize PHI exposure (this is currently an open design gate)

### 4.3 Resend Email Data Flow

**Concern:** Email content transits through Resend's infrastructure, which may not be in Canada.

**Required Controls:**
- Data minimization in email content (no phone numbers, no internal IDs)
- BAA/DPA must be executed with Resend before production launch
- Email content should include only: service name, date/time, therapist name, manage-booking link, Meet link (for online)
- Recipient email addresses are transmitted to Resend (necessary for delivery) but not stored long-term

#### Acceptance Checks
- [ ] Supabase project is deployed in a Canadian region (verified in project settings)
- [ ] Vercel deployment is configured for Canadian region preference
- [ ] BAA/DPA executed with Resend before production launch with PHI-adjacent data
- [ ] Google Calendar event content is minimized (design gate resolved for event title/description format)
- [ ] No client phone numbers are included in any data sent to Google or Resend
- [ ] Data residency configuration is documented and verified before first production deployment

---

## 5. Audit Trail Requirements

### 5.1 booking_audit_log Detailed Requirements

#### Required Fields Per Entry
- `entity_type`: The type of entity being audited
- `entity_id`: The UUID of the affected entity
- `action`: Dot-notation action code (e.g., `booking.created`)
- `actor`: Therapist UUID, `client:{sha256(email)}`, or `system`
- `before_state`: Sanitized previous state (for updates) or NULL (for creates)
- `after_state`: Sanitized new state or NULL (for deletes)
- `metadata`: Additional non-PHI context (error codes, scopes, sync status)
- `created_at`: UTC timestamp

#### Sanitization Rules for before_state / after_state

**MUST strip these fields before writing to audit log:**
- `client_name`
- `client_email`
- `client_phone`
- `cancellation_reason`
- `google_oauth_access_token`
- `google_oauth_refresh_token`
- `manage_token`

**SAFE to include in audit log state:**
- `id` (entity UUID)
- `status` (booking status)
- `start_time` / `end_time` (timestamps)
- `service_id`
- `therapist_id`
- `modality`
- `created_by` (actor type, not PHI)
- `is_active` (for services)
- `day_of_week`, `start_time`, `end_time` (for availability -- these are non-PHI schedule data)
- `sync_status`, `google_event_id` (for calendar events)
- `notification_type`, `status` (for notifications)

### 5.2 Immutability Enforcement

#### Requirements
- No UPDATE or DELETE operations permitted on booking_audit_log
- Enforcement at database trigger level (not just RLS)
- Application code must not attempt UPDATE/DELETE on audit log

#### Technical Controls (already defined in db-migration-plan.yaml)
- `prevent_audit_log_mutation()` trigger function raises exception on UPDATE or DELETE
- Triggers `audit_log_no_update` and `audit_log_no_delete` are applied BEFORE UPDATE/DELETE

#### Acceptance Checks
- [ ] UPDATE on booking_audit_log raises an exception (verified via test)
- [ ] DELETE on booking_audit_log raises an exception (verified via test)
- [ ] No application code path attempts UPDATE or DELETE on booking_audit_log

### 5.3 Audit Log Retention

- Minimum retention: 10 years (aligns with PHIPA health record retention for Ontario)
- Audit logs must not be purged before the corresponding data retention period expires
- Backup strategy must include audit logs with same retention period

### 5.4 No PHI/PII in Audit Log Payloads

This is a critical requirement that must be verified in every code review and compliance audit.

#### Acceptance Checks
- [ ] Audit log entries verified to contain no plaintext client names
- [ ] Audit log entries verified to contain no plaintext email addresses
- [ ] Audit log entries verified to contain no phone numbers
- [ ] Audit log entries verified to contain no cancellation reasons
- [ ] Audit log entries verified to contain no OAuth tokens
- [ ] Audit log entries verified to contain no manage_tokens
- [ ] Code review gate: any function that writes to booking_audit_log must be reviewed for PHI leakage

---

## 6. Business Associate Agreements (BAA)

### 6.1 Third-Party Services Handling PHI-Adjacent Data

| Service | Data Shared | BAA/DPA Status | Required Before |
|---|---|---|---|
| **Supabase** | All database contents (PHI-adjacent) | **Required** -- must verify Supabase BAA is signed | Production deployment |
| **Resend** | Client email addresses, booking details in email content | **Required** -- BAA/DPA with Resend | Production deployment |
| **Google Workspace** (Calendar API) | Client name, appointment time (in calendar events) | **Covered** by Google Workspace BAA (therapist's own account) | Therapist setup (therapist's responsibility for their own Google Workspace BAA) |
| **Vercel** | Application data in transit (server-side rendering may process PHI-adjacent data) | **Required** -- verify Vercel DPA | Production deployment |

#### Acceptance Checks
- [ ] BAA signed with Supabase (verified and documented)
- [ ] BAA/DPA signed with Resend (verified and documented)
- [ ] DPA signed with Vercel (verified and documented)
- [ ] Google Workspace BAA responsibility documented (therapist's own obligation for their Google account)

---

## 7. Breach Notification

### 7.1 Requirements
- HIPAA: Notify affected individuals within 60 days of discovery; notify HHS; notify media if >500 individuals
- PHIPA: Notify affected individuals at first reasonable opportunity; notify Information and Privacy Commissioner of Ontario
- PIPEDA: Notify affected individuals and Privacy Commissioner of Canada as soon as feasible

### 7.2 Detection (v0 Scope)
- Monitor for unusual access patterns (manual review of audit logs in v0)
- Automated alerting is a post-v0 enhancement
- Application error monitoring (Vercel/Supabase dashboards)

### 7.3 Response Plan
1. Contain: Revoke compromised credentials, disable affected accounts
2. Investigate: Review audit logs to determine scope
3. Notify: Per regulatory timelines above
4. Remediate: Fix vulnerability, rotate keys, update access controls
5. Document: Record breach in incident log

#### Acceptance Checks
- [ ] Breach notification process is documented (this document serves as initial documentation)
- [ ] Audit logs are available for breach investigation
- [ ] A process exists to revoke OAuth tokens and disable accounts in an emergency

---

## 8. Feature-Level Compliance Checklist

### 8.1 therapist-setup

**PHI/PII flows:** Therapist PII (name, email), OAuth tokens (sensitive-internal)

| Check | Implementer Responsibility | Auditor Verification |
|---|---|---|
| Google OAuth tokens encrypted with field-level encryption | Use Supabase Vault or pgsodium for token columns | Verify token columns are encrypted beyond TDE |
| OAuth tokens never logged | Ensure no console.log, debug output, or error handler logs token values | Search codebase for token variable names in log statements |
| Setup completion emits audit entry | Call audit log insert on setup completion | Verify audit log entry exists after setup |
| Google Calendar connection emits audit entry with scopes | Log scopes granted (non-PHI) | Verify audit entry includes scopes |
| RLS enforces own-row access | auth.uid() = therapists.id in all RLS policies | Attempt cross-tenant access and verify denial |
| Session timeout configured | Set Supabase session expiry | Verify inactive sessions expire |

### 8.2 booking-page

**PHI/PII flows:** Client PII collected (name, email, phone), consent recorded, PHI-adjacent booking created

| Check | Implementer Responsibility | Auditor Verification |
|---|---|---|
| Consent checkbox unchecked by default | Render checkbox without default checked state | Inspect DOM/source to verify no default checked |
| Consent language states purpose clearly | Include clear text about data collection and purpose | Review consent text for PIPEDA compliance |
| Client phone never logged | No logging of client_phone in server actions or API routes | Grep codebase for phone-related logging |
| Booking creation emits audit entry | Insert audit log with actor=client:{hash} | Verify audit log after booking |
| No therapist PII exposed on booking page | Only display practice_name, service info, availability | Inspect API responses for leaked therapist email/phone/ID |
| Slot lock does not store client PII | locked_by uses session ID, not email/name | Inspect slot_lock records |
| Data minimization verified | No unnecessary data in API responses | Review all booking page API responses |

### 8.3 booking-management

**PHI/PII flows:** PHI-adjacent booking data accessed and modified by therapist and client

| Check | Implementer Responsibility | Auditor Verification |
|---|---|---|
| Reschedule emits audit entry with before/after | Log sanitized before/after state | Verify audit entry has no PHI in state fields |
| Cancel emits audit entry | Log cancellation event | Verify audit entry exists |
| manage_token scoped to single booking | RLS or application logic restricts token to one booking | Attempt to use token for different booking |
| manage_token is cryptographically random | Use crypto.randomBytes or equivalent (min 32 bytes) | Verify entropy of generated tokens |
| Therapist manual create records correct actor | created_by = therapist UUID, audit actor = therapist UUID | Verify audit log actor for manual bookings |
| Client self-service does not expose other bookings | Token-based access returns only matched booking | Attempt enumeration via modified token |

### 8.4 booking-notifications

**PHI/PII flows:** Client email sent to Resend, booking details in email content

| Check | Implementer Responsibility | Auditor Verification |
|---|---|---|
| No client_phone in any email template | Exclude phone from all email templates | Review all email templates |
| No internal IDs in email content | Use human-readable identifiers only | Review email templates for UUID exposure |
| notification_log uses hashed email | SHA-256 hash of recipient email, never plaintext | Inspect notification_log records |
| Email sending logged in audit trail | Audit entry for notification.sent/failed | Verify audit entries exist |
| Cancelled booking reminders suppressed | Check booking status before sending reminder | Cancel a booking and verify no reminder fires |
| Data minimization in email content | Only service name, date/time, therapist name, manage link, Meet link | Review rendered email content |
| Resend API calls use HTTPS | All Resend SDK calls over TLS | Verify SDK configuration |

### 8.5 google-calendar-sync

**PHI/PII flows:** Client name and appointment time sent to Google Calendar API (data leaves Canada)

| Check | Implementer Responsibility | Auditor Verification |
|---|---|---|
| Calendar event content minimized | Limit event title/description to minimum necessary info | Review calendar event creation payload |
| No client_phone in calendar events | Exclude phone from all event data | Inspect event creation API call |
| Every calendar write is audit-logged | Audit entry for gcal.event.created/updated/deleted | Verify audit entries for all calendar operations |
| No silent writes | Calendar operations only triggered by booking actions | Review code for any autonomous calendar writes |
| OAuth token refresh handled securely | Token refresh does not log tokens | Verify no token logging during refresh |
| Sync failure does not expose PHI in error logs | Error messages contain only status codes and sync_status | Review error handling code |
| OAuth tokens encrypted at rest | Field-level encryption for token columns | Verify encryption configuration |

### 8.6 booking-services

**PHI/PII flows:** Minimal -- service data is public by design (therapist publishes service catalog)

| Check | Implementer Responsibility | Auditor Verification |
|---|---|---|
| RLS enforces tenant isolation | therapist_id = auth.uid() on all policies | Attempt cross-tenant access |
| All CRUD operations audit-logged | Audit entry for service.created/updated/deleted | Verify audit entries |
| Public read limited to active services | Anon RLS policy uses is_active = true | Verify inactive services not visible on booking page |
| Encryption at rest | Supabase TDE covers service data | Verify Supabase encryption settings |

### 8.7 availability

**PHI/PII flows:** None -- availability is non-PHI schedule configuration data

| Check | Implementer Responsibility | Auditor Verification |
|---|---|---|
| RLS enforces tenant isolation | therapist_id = auth.uid() on all policies | Attempt cross-tenant access |
| All CRUD operations audit-logged | Audit entry for availability.created/updated/deleted | Verify audit entries |
| Public read is acceptable | Availability data is non-PHI (days/hours only) | Confirm no PHI in availability records |
| Encryption at rest | Supabase TDE covers availability data | Verify Supabase encryption settings |

---

## 9. Compliance Concerns and Open Items

### 9.1 Open Design Gates with Compliance Impact

| Design Gate | Compliance Impact | Risk if Unresolved |
|---|---|---|
| Google Calendar event title/description format (google-calendar-sync Q1) | PHI minimization -- client name in calendar title is PHI-adjacent | **HIGH**: Excessive PHI in calendar events visible to anyone with calendar access |
| manage_token expiry policy (bookings.manage_token_expires_at) | Token lifespan affects risk of unauthorized access to PHI-adjacent data | **MEDIUM**: Long-lived tokens increase breach surface |
| Consent text content (booking-page Q4) | PIPEDA requires clear purpose statement | **HIGH**: Non-compliant consent language invalidates consent |
| Session timeout duration | HIPAA automatic logoff requirement | **MEDIUM**: No timeout allows indefinite access |

### 9.2 Pre-Production Blockers

These items must be resolved before any real client data is processed:

1. **BAAs must be signed** with Supabase, Resend, and Vercel
2. **Supabase region must be verified** as Canadian
3. **Field-level encryption** for OAuth tokens must be configured (Supabase Vault or pgsodium)
4. **Consent language** must be reviewed and finalized
5. **Calendar event format** design gate must be resolved with PHI minimization guidance

### 9.3 Items Acceptable for v0 Launch (with Backlog)

These items are acceptable for initial launch but must be addressed before significant data accumulates:

1. Automated data retention/deletion (manual process acceptable initially)
2. Automated breach detection/alerting (manual audit log review acceptable initially)
3. Formal Subject Access Request workflow (manual process via therapist acceptable initially)
4. Multi-factor authentication for therapists (Google OAuth provides single factor; MFA is a Google account setting, not Curio's responsibility in v0)

---

## References

- Security & Compliance Baseline: `.ops/security-compliance-baseline.md`
- PRD: `.ops/build/v0/prd.md`
- System Design: `.ops/build/system-design.yaml`
- DB Migration Plan: `.ops/build/v0/db-migration-plan.yaml`
- Feature Specs: `.ops/build/v0/{feature}/specs.md`
- Compliance Patterns: `.claude/skills/compliance-patterns/SKILL.md`
