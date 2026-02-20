# Booking Notifications -- Specs

## Goal

Send automated email notifications to clients and therapists for booking lifecycle events (confirmation, rescheduling, cancellation) and appointment reminders (day before, 1 hour before). This ensures both parties are informed without manual communication effort.

## Non-Goals

- No customizable reminder timing (fixed at day-before and 1-hour-before in v0; customization is a nice-to-have)
- No SMS or push notifications (email only in v0)
- No in-app notifications
- No email template customization by the therapist
- No marketing or promotional emails
- No digest or summary emails
- No AI-generated email content
- No client reply handling (notifications are outbound only)
- Human oversight is not required for automated booking-triggered emails (these are system-triggered, not AI-generated; they follow deterministic templates per PRD scope)

## Requirements

- R1: The system must send a booking confirmation email to the client immediately after a booking is created (via public page or therapist manual creation).
- R2: The system must send a booking confirmation email (or notification) to the therapist when a new booking is created.
- R3: The system must send a rescheduling notification email to the client when a booking is rescheduled (by therapist or client).
- R4: The system must send a rescheduling notification email to the therapist when a client reschedules.
- R5: The system must send a cancellation notification email to the client when a booking is cancelled (by therapist or client).
- R6: The system must send a cancellation notification email to the therapist when a client cancels.
- R7: The system must send an appointment reminder email to the client one day (24 hours) before the appointment.
- R8: The system must send an appointment reminder email to the client one hour before the appointment.
- R9: All emails must be sent via Resend (the designated email provider per PRD technical assumptions).
- R10: Email content must include relevant booking details: service name, date, time (in therapist's timezone), therapist name. For online appointments, the Google Meet link must be included in confirmation and reminder emails.
- R11: Confirmation and reminder emails must include a link for the client to manage their booking (reschedule/cancel).
- R12: Emails must not contain unnecessary PHI/PII beyond what is needed for the notification purpose (data minimization). Specifically: no client phone number in emails, no internal booking IDs exposed.
- R13: Email sending events must be logged in the audit trail (recipient email hash or ID, email type, timestamp, booking reference). Do not log the full email body or client email in plaintext in audit logs.
- R14: Reminder emails must not be sent for cancelled bookings. If a booking is cancelled after the reminder is scheduled but before it fires, the reminder must be suppressed.
- R15: All email infrastructure and data must comply with Canada data residency requirements.
- R16: Email templates must be accessible (proper HTML structure, alt text, sufficient contrast for email clients that support it).
- R17: Emails must clearly identify Curio as the sender and include the therapist's practice name.
- R18: Failed email delivery must be logged but must not block or roll back the booking operation. Email delivery is best-effort with retry. (Requires design gate: define retry strategy.)

## Acceptance Criteria (Scenarios)

### S1: Client receives confirmation email after booking
**Given** a client completes a booking for "Initial Consultation" on Monday at 10:00 AM
**When** the booking is confirmed
**Then** the client receives an email with: service name, date/time in therapist's timezone, therapist name, and a link to manage the booking

### S2: Therapist receives notification of new booking
**Given** a client books an appointment
**When** the booking is confirmed
**Then** the therapist receives an email notification with the client's name, service, and appointment date/time

### S3: Online appointment confirmation includes Meet link
**Given** a client books an online service
**When** the confirmation email is sent
**Then** the email includes the Google Meet link for the appointment

### S4: Client receives rescheduling notification
**Given** a therapist reschedules a booking from Monday 10:00 AM to Wednesday 2:00 PM
**When** the reschedule is confirmed
**Then** the client receives an email showing the old time (crossed out or clearly indicated) and the new time

### S5: Therapist receives notification when client reschedules
**Given** a client reschedules their booking via the manage-booking link
**When** the reschedule is confirmed
**Then** the therapist receives an email notification with the old and new appointment times

### S6: Client receives cancellation notification
**Given** a therapist cancels a booking
**When** the cancellation is processed
**Then** the client receives an email confirming the cancellation with the original appointment details

### S7: Day-before reminder is sent
**Given** a confirmed booking exists for Tuesday at 2:00 PM
**When** it is Monday at 2:00 PM (24 hours before)
**Then** the client receives a reminder email with appointment details and the manage-booking link

### S8: One-hour-before reminder is sent
**Given** a confirmed booking exists for Tuesday at 2:00 PM
**When** it is Tuesday at 1:00 PM (1 hour before)
**Then** the client receives a reminder email with appointment details

### S9: Reminders are suppressed for cancelled bookings
**Given** a booking for Wednesday at 10:00 AM is cancelled on Monday
**When** the scheduled reminder times arrive (Tuesday 10:00 AM and Wednesday 9:00 AM)
**Then** no reminder emails are sent for the cancelled booking

### S10: Email delivery failure does not block booking
**Given** a client completes a booking but the email service is temporarily unavailable
**When** the booking is confirmed
**Then** the booking is saved and confirmed in the system, the email delivery failure is logged, and delivery is retried

### S11: Data minimization in emails
**Given** a confirmation email is generated
**When** the email content is rendered
**Then** the email does not contain the client's phone number or any internal system identifiers

## Open Questions

- Q1: What is the email retry strategy for failed deliveries? How many retries, with what backoff, and is there a dead-letter mechanism? (Requires design gate.)
- Q2: Should therapist notification emails be configurable (e.g., opt out of new booking notifications)? The PRD does not mention therapist notification preferences. (Requires design gate.)
- Q3: What is the exact sender address format? (e.g., "Curio <bookings@curio.health>" or "{Therapist Practice} via Curio <noreply@curio.health>") (Requires design gate.)
- Q4: Should reminder emails include a "Cancel" link, or only a general "Manage booking" link? (Requires design gate.)
