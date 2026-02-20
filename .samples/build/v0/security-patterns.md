# Curio v0 -- Security Patterns

**Build version:** v0
**Created:** 2026-02-06
**Source artifacts:** security-compliance-baseline.md, system-design.yaml, db-migration-plan.yaml, all v0 feature specs
**Compliance baseline:** HIPAA/PHIPA from day 0, Canada data residency, zero trust, least privilege, fail-closed

---

## Table of Contents

1. [Authentication and Authorization Patterns](#1-authentication-and-authorization-patterns)
2. [Data Protection Patterns](#2-data-protection-patterns)
3. [Input Validation and Sanitization](#3-input-validation-and-sanitization)
4. [API Security](#4-api-security)
5. [Google API Security](#5-google-api-security)
6. [Security Checklist by Feature](#6-security-checklist-by-feature)
7. [Threat Model Summary](#7-threat-model-summary)

---

## 1. Authentication and Authorization Patterns

### 1.1 Google OAuth Flow (Therapist Authentication)

#### Pattern

Google OAuth is the sole identity provider. Authentication uses Supabase Auth with Google as the OAuth provider. Incremental authorization is required: initial sign-in requests `email` and `profile` scopes only; calendar scopes are requested separately at the Google Calendar connection step during setup.

#### Requirements

- Initial OAuth request must use only `email` and `profile` scopes
- Calendar scopes (`https://www.googleapis.com/auth/calendar.events`, `https://www.googleapis.com/auth/calendar.readonly`) requested only at the Calendar connection step
- Meet link generation requires `https://www.googleapis.com/auth/calendar.events` (conferenceData support)
- OAuth callback URL must be pinned to a single, known origin (no wildcard or dynamic callback URLs)
- OAuth state parameter must be used to prevent CSRF on the OAuth callback
- OAuth PKCE flow is required (Supabase Auth supports this by default)
- On OAuth error or denial, fail-closed: do not create a partial session, redirect to error page
- Google ID (`google_id`) mapped to Supabase Auth user ID; one Curio account per Google identity (enforced by UNIQUE constraint on `therapists.google_id`)

#### Acceptance Checks

- [ ] Initial OAuth flow requests only `email` and `profile` scopes
- [ ] Calendar scopes are not requested until the Calendar connection step
- [ ] OAuth state parameter is validated on callback to prevent CSRF
- [ ] PKCE is used for the OAuth flow
- [ ] OAuth callback URL is a fixed, server-side route (not client-configurable)
- [ ] Denying OAuth permissions results in a clear error, not a partial session
- [ ] Duplicate Google ID cannot create a second therapist account

### 1.2 Supabase Auth Session Management

#### Pattern

Supabase Auth manages JWT-based sessions. Server Components and Route Handlers extract the session via `createServerComponentClient` or `createRouteHandlerClient`. The user ID is always derived from `session.user.id`, never from request parameters, body, or headers.

#### Requirements

- All authenticated routes must verify the session before proceeding
- User ID must come from `session.user.id` exclusively
- Session tokens (JWTs) must not be logged or exposed in client-side JavaScript beyond what Supabase Auth provides
- Expired or invalid sessions must return 401
- Service role key (`SUPABASE_SERVICE_ROLE_KEY`) must never be exposed to the client or prefixed with `NEXT_PUBLIC_`
- Supabase anon key is safe for client use (it respects RLS)

#### Acceptance Checks

- [ ] All authenticated endpoints verify session before processing
- [ ] User ID is derived from session.user.id, never from request body or URL params
- [ ] Requests with expired sessions return 401
- [ ] Requests with no session to authenticated endpoints return 401
- [ ] `SUPABASE_SERVICE_ROLE_KEY` is not exposed in any client-side bundle
- [ ] No `NEXT_PUBLIC_SUPABASE_SERVICE_ROLE_KEY` variable exists anywhere

### 1.3 RLS Policy Patterns by Table

#### Pattern

Every table has RLS enabled. Policies enforce tenant isolation at the database level. Application-level authorization is defense-in-depth but never the sole access control.

#### Requirements

| Table | Authenticated (therapist) | Anonymous (client/public) | Service Role (system) |
|---|---|---|---|
| `therapists` | SELECT/UPDATE/INSERT own row (`id = auth.uid()`) | None | Full access for system operations |
| `services` | Full CRUD on own (`therapist_id = auth.uid()`) | SELECT active services only (`is_active = true`) | Full access |
| `availability` | Full CRUD on own (`therapist_id = auth.uid()`) | SELECT all (non-PHI schedule data) | Full access |
| `bookings` | SELECT/UPDATE own (`therapist_id = auth.uid()`) | INSERT (public booking); SELECT/UPDATE via `manage_token` | Full access |
| `booking_audit_log` | SELECT own entity logs | None | INSERT only (immutable -- UPDATE/DELETE blocked by trigger) |
| `slot_locks` | SELECT own | SELECT all (for slot computation); INSERT (for locking) | DELETE expired locks |
| `google_calendar_events` | SELECT own | None | INSERT/UPDATE (sync operations) |
| `notification_log` | SELECT own | None | INSERT/UPDATE (notification dispatch) |

#### Acceptance Checks

- [ ] RLS is enabled on all 8 tables
- [ ] Therapist cannot read or modify another therapist's data on any table
- [ ] Anonymous users can only read active services and availability (non-PHI data)
- [ ] Anonymous users can create bookings and slot locks (scoped to valid therapist via FK)
- [ ] Client manage_token access is scoped to a single booking only
- [ ] Audit log has no UPDATE or DELETE policies; trigger blocks mutation
- [ ] google_calendar_events and notification_log are not directly writable from the client

### 1.4 Token-Based Client Access for Booking Management

#### Pattern

Clients access their bookings via a `manage_token` -- a cryptographically random, unique, unguessable token embedded in email links. No client account or login is required. The token scopes access to exactly one booking.

#### Requirements

- `manage_token` must be generated using a cryptographically secure random generator (e.g., `crypto.randomUUID()` or `crypto.randomBytes(32).toString('hex')`)
- Token must be at least 32 characters (128 bits of entropy) to prevent brute-force
- Token must be unique per booking (enforced by UNIQUE constraint on `bookings.manage_token`)
- Token must be transmitted only via HTTPS (TLS in transit)
- Token must not appear in server logs, audit logs, or error messages
- Token-based access must be limited to: viewing booking details, rescheduling, and cancelling the specific booking
- Token must not grant access to any other booking, therapist data, or system data
- Token expiry policy must be defined (design gate noted in system-design.yaml -- recommend: token valid until 24 hours after appointment end time)
- Manage-booking URLs must not be indexable by search engines (add `noindex` meta tag, exclude from sitemap)

#### Acceptance Checks

- [ ] manage_token is at least 32 characters of cryptographically random data
- [ ] manage_token is unique per booking (DB constraint)
- [ ] Token grants access to exactly one booking -- attempting to modify the token to access another booking fails
- [ ] Token is not logged in audit logs or server logs
- [ ] Token-based pages include `noindex` meta tag
- [ ] Expired tokens (if expiry is implemented) return a clear error, not a redirect to another booking

---

## 2. Data Protection Patterns

### 2.1 PHI/PII Handling Rules

#### Pattern

All booking data containing client information (name, email, phone) is classified as PHI-adjacent under HIPAA/PHIPA. Data minimization is mandatory: collect only what is needed, store only what is necessary, expose only what is required for the specific operation.

#### Requirements

**Data classified as PHI/PII (protected):**
- Client name (`bookings.client_name`)
- Client email (`bookings.client_email`)
- Client phone (`bookings.client_phone`)
- Therapist email (`therapists.email`)
- Therapist full name (`therapists.full_name`)
- OAuth tokens (`therapists.google_oauth_access_token`, `therapists.google_oauth_refresh_token`)
- Booking manage token (`bookings.manage_token`)

**Data classified as non-PHI (safe for public access):**
- Service names, durations, modalities
- Availability time blocks (day of week, start/end times)
- Therapist practice name
- Therapist timezone
- Booking slug

**Data minimization rules:**
- Client phone must never appear in emails, logs, calendar events, or API responses to unauthorized parties
- Internal booking IDs must not appear in client-facing emails
- Audit logs must use hashed emails (`client:{sha256(email)}`) for client actor identification, never plaintext email
- Notification log stores `recipient_email_hash`, never plaintext email
- Google Calendar event titles must not contain full client name (design gate: recommend "Client Appointment" or first name only)
- Email content must not contain phone numbers or internal system IDs

#### Acceptance Checks

- [ ] Client phone number is never included in any email notification
- [ ] Client phone number is never included in Google Calendar event details
- [ ] Audit log actor field for clients uses hashed email, not plaintext
- [ ] notification_log.recipient_email_hash is a hash, not plaintext email
- [ ] No internal booking UUIDs appear in client-facing emails
- [ ] Google Calendar event title does not contain the client's full name
- [ ] Public booking page API responses do not expose therapist email or phone
- [ ] API error responses do not leak internal IDs, stack traces, or PHI

### 2.2 Encryption Requirements

#### Pattern

Encryption at rest and in transit is mandatory for all data. Field-level encryption is required for OAuth tokens and other high-sensitivity secrets.

#### Requirements

**In transit:**
- TLS 1.2+ on all connections (Vercel enforces HTTPS by default)
- HSTS header must be set
- No mixed content (HTTP resources on HTTPS pages)

**At rest:**
- Supabase Postgres encrypts all data at rest by default (AES-256)
- OAuth tokens (`google_oauth_access_token`, `google_oauth_refresh_token`) must have additional field-level encryption via Supabase Vault or application-level encryption before storage
- Encryption keys must be managed via Supabase Vault or environment-variable-based KMS, never hardcoded

**Encryption key management:**
- Token encryption key must be stored as an environment variable (`OAUTH_TOKEN_ENCRYPTION_KEY`) or in Supabase Vault
- Key must never appear in code, logs, or client bundles
- Key rotation strategy must be defined before production (can be deferred beyond v0 launch but must be documented)

#### Acceptance Checks

- [ ] All endpoints served over HTTPS (TLS 1.2+)
- [ ] HSTS header is present on all responses
- [ ] OAuth access and refresh tokens are encrypted before storage in the database (not just Postgres-level encryption)
- [ ] Token encryption key is stored in environment variables or Supabase Vault, not in code
- [ ] Decrypted tokens are never logged or included in error messages
- [ ] Supabase project is configured in Canada region

### 2.3 Audit Logging

#### Pattern

All state-changing operations emit an audit log entry to `booking_audit_log`. The audit log is append-only (immutable). PHI is minimized in log entries.

#### Requirements

**What to log:**
- Entity type, entity ID, action name
- Actor (therapist UUID, `client:{email_hash}`, or `system`)
- Before/after state (for updates)
- Timestamp
- Metadata (scopes granted, error context -- non-PHI only)

**What NOT to log:**
- Plaintext email addresses
- Phone numbers
- OAuth tokens or secrets
- Full email bodies
- manage_token values
- Passwords or credentials of any kind

**Actions that require audit entries:**
- `setup.completed`
- `google.oauth.connected`, `google.oauth.revoked`
- `service.created`, `service.updated`, `service.deleted`
- `availability.created`, `availability.updated`, `availability.deleted`
- `booking.created`, `booking.rescheduled`, `booking.cancelled`
- `gcal.event.created`, `gcal.event.updated`, `gcal.event.deleted`, `gcal.event.failed`
- `notification.sent`, `notification.failed`, `notification.suppressed`

#### Acceptance Checks

- [ ] Every state-changing operation emits an audit log entry
- [ ] Audit log entries never contain plaintext email, phone, tokens, or secrets
- [ ] Audit log INSERT operations succeed; UPDATE and DELETE operations are blocked by database trigger
- [ ] Actor field uses therapist UUID for therapist actions and hashed email for client actions
- [ ] before_state and after_state do not contain OAuth tokens or manage_token values

---

## 3. Input Validation and Sanitization

### 3.1 Server-Side Validation

#### Pattern

All input is validated on the server side using Zod schemas before any database operation. Client-side validation is for UX only and must never be relied upon for security. The Supabase client library with parameterized queries prevents SQL injection.

#### Requirements

**Booking form (public, anonymous):**

| Field | Validation | Max Length |
|---|---|---|
| `client_name` | Non-empty string, trimmed, no HTML tags | 200 characters |
| `client_email` | Valid email format (RFC 5322) | 254 characters |
| `client_phone` | Valid phone format (digits, spaces, hyphens, parentheses, plus sign) | 20 characters |
| `client_consent` | Must be `true` (boolean) | N/A |
| `service_id` | Valid UUID, must reference an active service for the therapist | 36 characters |
| `start_time` | Valid ISO 8601 datetime, must be in the future, must respect minimum notice | N/A |

**Service CRUD (authenticated):**

| Field | Validation | Max Length |
|---|---|---|
| `name` | Non-empty string, trimmed | 100 characters |
| `description` | Optional string, trimmed | 500 characters |
| `duration_minutes` | Integer, >= 15, <= 480 (8 hours) | N/A |
| `modality` | Enum: `online` or `in_person` | N/A |

**Availability (authenticated):**

| Field | Validation | Max Length |
|---|---|---|
| `day_of_week` | Integer, 0-6 | N/A |
| `start_time` | Valid time format (HH:MM) | N/A |
| `end_time` | Valid time format (HH:MM), must be > start_time | N/A |

**Therapist setup (authenticated):**

| Field | Validation | Max Length |
|---|---|---|
| `practice_name` | Non-empty string, trimmed | 200 characters |
| `timezone` | Must be a valid IANA timezone identifier | 50 characters |
| `slug` | Lowercase alphanumeric + hyphens, no leading/trailing hyphens, unique | 50 characters |

#### Acceptance Checks

- [ ] All server endpoints validate input with Zod schemas before database operations
- [ ] Invalid input returns 400 with a descriptive (but non-leaking) error message
- [ ] String fields have maximum length limits enforced server-side
- [ ] Enum fields reject values outside the allowed set
- [ ] UUID fields are validated for format before use in queries
- [ ] Email format validation uses a standard library, not a custom regex
- [ ] Booking start_time is validated to be in the future and respect minimum_notice_hours
- [ ] Slug validation prevents special characters, SQL injection, and path traversal

### 3.2 XSS Prevention

#### Pattern

React/Next.js escapes output by default. The risk is low but must be managed for user-provided content displayed on the booking page (therapist practice name, service names/descriptions).

#### Requirements

- Never use `dangerouslySetInnerHTML` with user-provided content
- Strip HTML tags from all user input on the server side before storage
- Content-Security-Policy (CSP) header must be set to restrict inline scripts
- X-Content-Type-Options: nosniff header must be set

#### Acceptance Checks

- [ ] No use of `dangerouslySetInnerHTML` with user-controlled data
- [ ] CSP header is configured and does not allow `unsafe-inline` for scripts
- [ ] X-Content-Type-Options: nosniff header is present
- [ ] User-provided text (practice name, service name, description) is sanitized before storage

### 3.3 SQL Injection Prevention

#### Pattern

All database queries use the Supabase client library with parameterized queries. No raw SQL string concatenation with user input.

#### Requirements

- All queries must use the Supabase JS client (`.from().select().eq()` pattern)
- If raw SQL is needed (Edge Functions, migrations), use parameterized queries only
- No string interpolation of user input into SQL
- Database CHECK constraints provide defense-in-depth for enum fields

#### Acceptance Checks

- [ ] No raw SQL string concatenation with user input anywhere in the codebase
- [ ] All database queries use the Supabase client or parameterized queries
- [ ] Database CHECK constraints enforce enum values (status, modality) as defense-in-depth

---

## 4. API Security

### 4.1 Edge Function and Server Action Authentication

#### Pattern

Mutations go through Next.js Server Actions or Supabase Edge Functions. Server Actions run on the server by default in Next.js App Router. Public endpoints (booking page, slot locking, booking creation) must still validate input even though they do not require authentication.

#### Requirements

**Authenticated endpoints (therapist dashboard):**
- All Server Actions for therapist operations must verify session via `createServerComponentClient`
- Edge Functions for system operations (GCal sync, notifications) must use `SUPABASE_SERVICE_ROLE_KEY` and validate the trigger source
- Service role operations must not be callable from the client

**Public endpoints (booking page):**
- Booking creation and slot locking must route through Server Actions or Edge Functions (not direct Supabase client inserts from the browser)
- Even though booking creation is anonymous, input must be fully validated
- Booking creation must verify: service_id references a valid active service, start_time is within availability, slot is not already booked or locked

#### Acceptance Checks

- [ ] All therapist mutation endpoints verify session before proceeding
- [ ] Public booking creation routes through a Server Action or Edge Function, not a direct client-side Supabase insert
- [ ] Edge Functions using service_role are not directly callable from client-side code
- [ ] Public endpoints validate all input even though authentication is not required

### 4.2 Rate Limiting

#### Pattern

Public-facing endpoints must be rate-limited to prevent abuse. Rate limiting is applied at the Edge (Vercel middleware) or application level. Authenticated endpoints have higher limits than anonymous endpoints.

#### Requirements

| Endpoint | Rate Limit | Window | Notes |
|---|---|---|---|
| Public booking page (GET) | 60 requests | 1 minute | Per IP |
| Slot lock creation (POST) | 10 requests | 1 minute | Per IP -- prevents slot lock flooding |
| Booking creation (POST) | 5 requests | 1 minute | Per IP -- prevents booking spam |
| Booking management via token (GET/PUT) | 20 requests | 1 minute | Per IP |
| Therapist dashboard (authenticated) | 120 requests | 1 minute | Per session |
| OAuth callback | 10 requests | 1 minute | Per IP -- prevents OAuth abuse |

#### Acceptance Checks

- [ ] Rate limiting is applied to all public-facing endpoints
- [ ] Slot lock creation is rate-limited to prevent flooding
- [ ] Booking creation is rate-limited to prevent spam
- [ ] Rate limit responses return 429 with Retry-After header
- [ ] Rate limiting does not block legitimate booking flows (limits are reasonable)

### 4.3 CSRF Protection

#### Pattern

Next.js Server Actions have built-in CSRF protection (they validate the Origin header). For any custom API routes, CSRF tokens must be validated. The OAuth flow uses the state parameter for CSRF protection.

#### Requirements

- Server Actions: rely on Next.js built-in CSRF protection (Origin header validation)
- Custom API routes (if any): validate CSRF token or Origin header
- OAuth callback: validate state parameter matches the one sent in the authorization request
- SameSite=Lax cookie attribute on session cookies (Supabase Auth default)

#### Acceptance Checks

- [ ] Server Actions are used for mutations (built-in CSRF protection)
- [ ] OAuth state parameter is validated on callback
- [ ] Session cookies have SameSite=Lax or SameSite=Strict attribute
- [ ] No custom API routes accept mutations without CSRF protection

### 4.4 Slot Lock Abuse Prevention

#### Pattern

Slot locks are temporary holds with a mandatory expiration. The system must prevent a single actor from locking many slots simultaneously to denial-of-service other clients.

#### Requirements

- Slot lock expiration must be short (recommended: 10 minutes, design gate pending)
- A single client session/IP should not be able to hold more than 3 active slot locks simultaneously
- Expired locks must be filtered out during slot computation (not relied upon for cleanup alone)
- Partial unique index on `(therapist_id, start_time, end_time) WHERE expires_at > now()` prevents duplicate locks on the same slot
- Application-level validation must complement the database constraint (the partial index with `now()` is best-effort)

#### Acceptance Checks

- [ ] Slot locks have a mandatory expiration time
- [ ] A single client cannot hold more than 3 active slot locks simultaneously
- [ ] Expired slot locks are excluded from slot availability computation
- [ ] Attempting to lock an already-locked slot returns a clear error
- [ ] Slot lock creation validates that the requested time falls within therapist availability

### 4.5 Security Headers

#### Pattern

Security headers must be set on all responses to mitigate common web attacks.

#### Requirements

The following headers must be present on all responses:

- `Strict-Transport-Security: max-age=31536000; includeSubDomains` (HSTS)
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY` (or SAMEORIGIN if iframing is needed)
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://*.supabase.co https://accounts.google.com;` (adjust as needed for dependencies)
- `Permissions-Policy: camera=(), microphone=(), geolocation=()`

#### Acceptance Checks

- [ ] HSTS header is present on all responses
- [ ] X-Content-Type-Options: nosniff is present
- [ ] X-Frame-Options is set to DENY
- [ ] Referrer-Policy is set to strict-origin-when-cross-origin
- [ ] Content-Security-Policy is configured and tested
- [ ] Permissions-Policy restricts unnecessary browser APIs

---

## 5. Google API Security

### 5.1 OAuth Scope Minimization

#### Pattern

OAuth scopes follow least-privilege. Scopes are requested incrementally: only what is needed at each step.

#### Requirements

**Step 1 -- Sign-in:**
- `openid`
- `email`
- `profile`

**Step 2 -- Calendar connection (incremental authorization):**
- `https://www.googleapis.com/auth/calendar.events` (create/update/delete events, includes Meet conference data)

**Scope restrictions:**
- Do not request `https://www.googleapis.com/auth/calendar` (full read/write to all calendars) -- use the more restricted `calendar.events` scope
- Do not request any Drive, Gmail, or other scopes
- Granted scopes must be stored in `therapists.google_oauth_scopes` for verification
- Before any Calendar API call, verify the required scope is in the stored scopes array

#### Acceptance Checks

- [ ] Initial sign-in requests only openid, email, and profile scopes
- [ ] Calendar connection requests only calendar.events scope (not full calendar access)
- [ ] No Drive, Gmail, or other unnecessary scopes are requested
- [ ] Granted scopes are stored and verified before Calendar API calls
- [ ] Scope changes require a new authorization flow (not silently added)

### 5.2 Token Refresh and Revocation Handling

#### Pattern

OAuth tokens are short-lived. The refresh token is used to obtain new access tokens. Token refresh is automatic and transparent. Token revocation is handled gracefully.

#### Requirements

- Access token refresh must happen server-side before Calendar API calls
- If the access token is expired, use the refresh token to obtain a new one
- If the refresh token is invalid (revoked by user at Google), mark `google_calendar_connected = false`
- On revocation, log an audit entry (`google.oauth.revoked`) and notify the therapist
- Token refresh errors must not expose tokens in error messages or logs
- Refreshed tokens must be re-encrypted before storage

#### Acceptance Checks

- [ ] Expired access tokens are automatically refreshed before Calendar API calls
- [ ] Token refresh happens server-side only, never client-side
- [ ] Revoked refresh tokens result in graceful degradation (calendar disconnected, bookings still work)
- [ ] Token revocation logs an audit entry
- [ ] Therapist is notified when calendar sync is disconnected due to revocation
- [ ] Refreshed tokens are encrypted before being stored back to the database
- [ ] Token refresh errors do not expose tokens in logs or error responses

### 5.3 Calendar API Data Flow

#### Pattern

Data sent to Google Calendar must follow data minimization. Calendar events must not expose excessive PHI. Calendar is not the source of truth for bookings.

#### Requirements

- Google Calendar event title: use a generic or minimal format (design gate -- recommend: "Appointment - {ServiceName}" without full client name)
- Google Calendar event description: include only service name and duration; no client phone, no internal IDs
- If client name is included in the event (design gate), use first name only
- Calendar event attendee list: do not add client as attendee (this would expose the therapist's calendar to the client via Google Calendar UI)
- Meet link: generated server-side as part of the Calendar event creation; never generate Meet links outside of a booking action
- API errors must not include OAuth tokens in error messages
- All Calendar API calls must be logged to `booking_audit_log` (action, booking_id, event_id, success/failure)

#### Acceptance Checks

- [ ] Calendar event title does not include client's full name
- [ ] Calendar event description does not include client phone number or internal IDs
- [ ] Client is not added as an attendee to the Google Calendar event
- [ ] Meet links are generated only for online-modality bookings
- [ ] Calendar API error handling does not leak OAuth tokens
- [ ] Every Calendar API call (create, update, delete) has a corresponding audit log entry

---

## 6. Security Checklist by Feature

### 6.1 Therapist Setup

**Developer must implement:**
- [ ] Google OAuth with PKCE and state parameter validation
- [ ] Incremental authorization (profile+email first, calendar scopes second)
- [ ] OAuth token encryption before storage (using Supabase Vault or application-level AES-256)
- [ ] RLS policy enforcement on therapists table (own-row access only)
- [ ] Slug validation: alphanumeric + hyphens, lowercase, no path traversal characters
- [ ] Timezone validation: must be a valid IANA timezone from a whitelist
- [ ] Audit log entries for setup.completed and google.oauth.connected
- [ ] Session verification on all setup endpoints

**Security auditor will verify:**
- [ ] OAuth tokens are encrypted at rest (not plaintext in DB)
- [ ] OAuth tokens never appear in logs, client responses, or error messages
- [ ] RLS prevents cross-therapist data access
- [ ] Slug input cannot be used for path traversal or XSS
- [ ] Audit entries are created for all state changes during setup

### 6.2 Booking Page

**Developer must implement:**
- [ ] Server-side input validation (Zod) for all booking form fields
- [ ] Rate limiting on booking creation and slot lock endpoints
- [ ] Slot lock with mandatory expiration and per-client limit
- [ ] Booking creation through Server Action (not direct client-side DB insert)
- [ ] manage_token generation using cryptographically secure random generator (>= 32 chars)
- [ ] Data minimization: public API does not expose therapist email, phone, or internal IDs
- [ ] Consent checkbox default unchecked, explicit affirmative action required
- [ ] Audit log entry for booking.created

**Security auditor will verify:**
- [ ] Booking page does not expose sensitive therapist data in HTML, API responses, or network requests
- [ ] Slot lock cannot be abused for denial of service (rate limit + per-client limit)
- [ ] Double-booking is prevented at database level (partial unique index + application checks)
- [ ] manage_token is cryptographically random and unguessable
- [ ] Input validation rejects malformed email, phone, and name values
- [ ] Booking creation fails cleanly if service is inactive or slot is unavailable

### 6.3 Booking Management

**Developer must implement:**
- [ ] Session verification for all therapist-side booking operations
- [ ] manage_token-based access for client-side operations (scoped to single booking)
- [ ] Token-based access cannot enumerate or access other bookings
- [ ] Rescheduling uses the same slot-lock mechanism as new bookings
- [ ] Cancellation releases the slot and does not delete the booking record
- [ ] Audit log entries for booking.rescheduled and booking.cancelled with before/after state
- [ ] RLS enforcement: therapist sees only own bookings
- [ ] noindex meta tag on manage-booking pages

**Security auditor will verify:**
- [ ] manage_token cannot be predicted or brute-forced
- [ ] Modifying the token in the URL does not grant access to other bookings
- [ ] Therapist RLS prevents cross-tenant booking access
- [ ] Rescheduling validates new slot availability before committing
- [ ] Cancelled booking data is retained (not deleted) for audit compliance

### 6.4 Google Calendar Sync

**Developer must implement:**
- [ ] Server-side only Calendar API calls (never from client)
- [ ] Automatic token refresh with re-encryption of new tokens
- [ ] Graceful degradation on token revocation (disconnect calendar, keep bookings working)
- [ ] Data minimization in calendar event content (no full client name, no phone, no internal IDs)
- [ ] Audit log entries for all Calendar API operations
- [ ] Retry with exponential backoff for transient API failures
- [ ] No Calendar writes outside of booking lifecycle actions

**Security auditor will verify:**
- [ ] OAuth tokens are never exposed in API responses, logs, or client bundles
- [ ] Calendar events do not contain excessive PHI
- [ ] Token refresh re-encrypts new tokens before storage
- [ ] Calendar API failures do not expose credentials in error messages
- [ ] No Calendar writes occur outside explicit booking actions

### 6.5 Booking Notifications

**Developer must implement:**
- [ ] Data minimization in email content: no phone numbers, no internal IDs
- [ ] Notification log stores hashed email (not plaintext) for recipient
- [ ] Email templates do not include manage_token in plaintext logs
- [ ] Resend API key stored as server-side environment variable only
- [ ] Reminder suppression for cancelled bookings
- [ ] Audit log entries for notification.sent, notification.failed, notification.suppressed

**Security auditor will verify:**
- [ ] Email content does not include client phone number or internal booking IDs
- [ ] notification_log.recipient_email_hash is a hash, not plaintext
- [ ] Email body is not stored in any log table
- [ ] Resend API key is not exposed in client bundles
- [ ] Cancelled booking reminders are properly suppressed

### 6.6 Booking Services

**Developer must implement:**
- [ ] Session verification on all CRUD endpoints
- [ ] Input validation (Zod) for name, duration, modality, description
- [ ] RLS enforcement: therapist CRUD on own services only
- [ ] Audit log entries for service.created, service.updated, service.deleted
- [ ] Service deletion does not break existing bookings (ON DELETE RESTRICT or soft-delete)

**Security auditor will verify:**
- [ ] Cross-therapist service access is denied by RLS
- [ ] Input validation prevents empty names, negative durations, invalid modalities
- [ ] Service names displayed on booking page are sanitized (no XSS)
- [ ] Audit log records before/after state for updates

### 6.7 Availability

**Developer must implement:**
- [ ] Session verification on all CRUD endpoints
- [ ] Input validation for day_of_week (0-6), time format, time ordering (end > start)
- [ ] Overlap validation at application level (complementing DB exclusion constraint)
- [ ] RLS enforcement: therapist CRUD on own availability only
- [ ] Audit log entries for availability changes
- [ ] Availability changes do not affect existing confirmed bookings

**Security auditor will verify:**
- [ ] Cross-therapist availability access is denied by RLS (write operations)
- [ ] Overlap validation works at both application and database levels
- [ ] Public read of availability data does not expose PHI (availability is non-PHI)
- [ ] Availability input validation prevents invalid time values

---

## 7. Threat Model Summary

### STRIDE Analysis

| Threat | Attack Vector | Mitigation | Status |
|---|---|---|---|
| **Spoofing** | Forged therapist identity | Google OAuth with PKCE, session verification, user ID from session only | Covered |
| **Spoofing** | Forged client identity | manage_token is unguessable (128+ bits entropy), scoped to single booking | Covered |
| **Tampering** | Modified booking data in transit | TLS 1.2+ on all connections, HTTPS enforced | Covered |
| **Tampering** | Direct DB manipulation bypassing app | RLS policies on all tables, service_role key server-only | Covered |
| **Tampering** | Audit log modification | Immutability trigger blocks UPDATE/DELETE on booking_audit_log | Covered |
| **Repudiation** | Therapist denies action | Audit log with therapist UUID, timestamp, before/after state | Covered |
| **Repudiation** | Client denies action | Audit log with hashed client email, timestamp | Covered |
| **Info Disclosure** | Cross-tenant data leak | RLS policies enforce tenant isolation on every table | Covered |
| **Info Disclosure** | PHI in logs | Audit log uses hashed emails; no phone/tokens/secrets in logs | Covered |
| **Info Disclosure** | PHI in Google Calendar | Data minimization in event content (design gate on title format) | Covered (needs design gate resolution) |
| **Info Disclosure** | OAuth token exposure | Tokens encrypted at rest, never logged, never sent to client | Covered |
| **Info Disclosure** | manage_token in search engines | noindex meta tag, excluded from sitemap | Covered |
| **DoS** | Slot lock flooding | Rate limit on slot lock creation + per-client active lock limit | Covered |
| **DoS** | Booking spam | Rate limit on booking creation (5/min per IP) | Covered |
| **DoS** | Large input payloads | Input length limits on all string fields | Covered |
| **Elevation of Privilege** | Client escalates to therapist | manage_token grants only booking view/reschedule/cancel, no other access | Covered |
| **Elevation of Privilege** | Therapist accesses other tenant | RLS policies with `auth.uid()` checks on all tables | Covered |
| **Elevation of Privilege** | Service role key leak | Key stored server-side only, never prefixed with NEXT_PUBLIC_ | Covered |

### Open Security Design Gates

The following items require design gate resolution before implementation. These do not block security pattern definition but must be resolved before the security-auditor sign-off:

1. **Calendar event title format** -- determines PHI exposure in Google Calendar. Recommend: "Appointment - {ServiceName}" without client full name.
2. **Slot lock timeout duration** -- affects DoS surface. Recommend: 10 minutes.
3. **manage_token expiry policy** -- determines how long client access persists. Recommend: 24 hours after appointment end.
4. **Phone format validation rules** -- Canadian only or international. Recommend: permissive international format with 7-20 digit validation.
5. **Maximum concurrent slot locks per client** -- affects DoS surface. Recommend: 3.

---

## Secrets Inventory

| Secret | Storage Location | Accessible To | Notes |
|---|---|---|---|
| `SUPABASE_SERVICE_ROLE_KEY` | Vercel env vars | Server-side only | Bypasses RLS -- never expose to client |
| `NEXT_PUBLIC_SUPABASE_URL` | Vercel env vars | Client + server | Safe for public access |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Vercel env vars | Client + server | Safe -- respects RLS |
| `GOOGLE_CLIENT_ID` | Vercel env vars | Server-side only | Used in OAuth flow |
| `GOOGLE_CLIENT_SECRET` | Vercel env vars | Server-side only | Never expose to client |
| `RESEND_API_KEY` | Vercel env vars | Server-side only | Email service API key |
| `OAUTH_TOKEN_ENCRYPTION_KEY` | Vercel env vars or Supabase Vault | Server-side only | For field-level encryption of OAuth tokens |

**Verification required before deployment:**
- [ ] `.gitignore` includes `.env`, `.env.*`, `.env.local`
- [ ] No secrets in committed code or config files
- [ ] No `NEXT_PUBLIC_` prefix on server-only secrets
- [ ] All secrets are set in Vercel project environment variables
- [ ] Supabase project is in Canada region

---

*Security patterns defined by security-engineer agent. Implementation by fullstack-developer, verification by security-auditor.*
