# Capability: Booking Notifications

Email notifications for booking events and reminders.

## ADDED Requirements

### Requirement: Booking Confirmation Email

The system SHALL send confirmation emails when bookings are created.

#### Scenario: Client confirmation email
- **WHEN** a booking is successfully created
- **THEN** the client receives an email containing:
  - Appointment date and time (in therapist's timezone)
  - Service name and duration
  - Therapist name and practice
  - Location or Google Meet link (for online appointments)
  - Reschedule and cancel links
  - ICS calendar attachment

#### Scenario: Therapist notification email
- **WHEN** a booking is created (by client or manually)
- **THEN** the therapist receives an email with:
  - Appointment date and time
  - Client name, email, and phone
  - Service name
  - Link to view booking in dashboard

### Requirement: Reschedule Notification Email

The system SHALL send notification emails when bookings are rescheduled.

#### Scenario: Client-initiated reschedule
- **WHEN** a client reschedules their booking
- **THEN** the client receives confirmation of the new time
- **AND** the therapist receives notification of the change

#### Scenario: Therapist-initiated reschedule
- **WHEN** a therapist reschedules a booking
- **THEN** the client receives an email with:
  - Original and new appointment times
  - Reason for reschedule (if provided)
  - Updated reschedule and cancel links
  - Updated ICS attachment

### Requirement: Cancellation Notification Email

The system SHALL send notification emails when bookings are cancelled.

#### Scenario: Client-initiated cancellation
- **WHEN** a client cancels their booking
- **THEN** the client receives cancellation confirmation
- **AND** the therapist receives notification with cancellation reason (if provided)

#### Scenario: Therapist-initiated cancellation
- **WHEN** a therapist cancels a booking
- **THEN** the client receives an email with:
  - Cancelled appointment details
  - Reason for cancellation
  - Link to book a new appointment

### Requirement: Appointment Reminders

The system SHALL send reminder emails before appointments.

#### Scenario: Day-before reminder
- **WHEN** an appointment is scheduled for the next day
- **THEN** the client receives a reminder email 24 hours before
- **AND** the email includes appointment details, Meet link (if online), and reschedule/cancel links

#### Scenario: One-hour reminder
- **WHEN** an appointment is in one hour
- **THEN** the client receives a reminder email
- **AND** the email prominently displays the Meet link (if online)

#### Scenario: Cancelled booking no reminders
- **WHEN** a booking has been cancelled
- **THEN** no reminder emails are sent for that booking

#### Scenario: Reminder timing
- **WHEN** reminders are scheduled
- **THEN** they are sent at approximately the specified time (24h before, 1h before)
- **AND** timing is based on the appointment time, not creation time

### Requirement: Email Delivery

The system SHALL ensure reliable email delivery.

#### Scenario: Email provider
- **WHEN** sending any booking-related email
- **THEN** the system uses Resend for email delivery

#### Scenario: Delivery tracking
- **WHEN** an email is sent
- **THEN** the delivery status is logged for troubleshooting

#### Scenario: Failed delivery handling
- **WHEN** an email fails to send
- **THEN** the system retries delivery with exponential backoff
- **AND** logs the failure for monitoring

### Requirement: Email Content Standards

The system SHALL maintain consistent, professional email content.

#### Scenario: Branding consistency
- **WHEN** sending any email
- **THEN** the email includes consistent Curio branding
- **AND** displays the therapist's practice name

#### Scenario: Mobile-friendly format
- **WHEN** a client views emails on mobile
- **THEN** the email is responsive and readable

#### Scenario: Plain text fallback
- **WHEN** an email client does not support HTML
- **THEN** a plain text version of the email is available

#### Scenario: Unsubscribe compliance
- **WHEN** sending reminder emails
- **THEN** include appropriate unsubscribe/manage preferences link
- **AND** respect any client communication preferences
