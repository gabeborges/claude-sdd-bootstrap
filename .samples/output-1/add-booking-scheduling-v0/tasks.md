# Tasks: Booking & Scheduling System (v0)

## 1. Project Setup & Infrastructure

- [x] 1.1 Initialize Next.js project with TypeScript, Tailwind CSS, shadcn
- [x] 1.2 Configure Supabase project (Canada region for data residency) - *Setup guide created*
- [x] 1.3 Set up Vercel deployment with environment variables - *Setup guide created*
- [x] 1.4 Configure Resend for transactional emails - *Setup guide created*
- [x] 1.5 Set up Google Cloud project with OAuth credentials and Calendar API - *Setup guide created*

## 2. Database Schema & Foundation

- [x] 2.1 Create `therapists` table with Google identity fields, timezone, profile
- [x] 2.2 Create `services` table with name, duration, location_type, ordering
- [x] 2.3 Create `availability_schedules` table for weekly recurring availability
- [x] 2.4 Create `availability_overrides` table for date-specific exceptions
- [x] 2.5 Create `clients` table with name, email, phone, consent tracking
- [x] 2.6 Create `bookings` table with status lifecycle, foreign keys, calendar event ID
- [x] 2.7 Create `slot_locks` table for double-booking prevention
- [x] 2.8 Create `audit_logs` table (append-only structure)
- [x] 2.9 Implement Row Level Security (RLS) policies for all tables
- [x] 2.10 Create database indexes for common query patterns

## 3. Authentication & Onboarding

- [x] 3.1 Implement Google OAuth sign-in flow with Supabase Auth
- [x] 3.2 Request and store calendar.events OAuth scope
- [x] 3.3 Build timezone selection UI for onboarding
- [x] 3.4 Build practice profile setup (name, title)
- [x] 3.5 Implement username/booking URL assignment
- [x] 3.6 Build onboarding completion checklist UI
- [x] 3.7 Implement setup completion gate for booking page activation

## 4. Services Management

- [x] 4.1 Build service creation form (name, duration, description, location type)
- [x] 4.2 Implement service list view with reordering (drag-and-drop)
- [x] 4.3 Build service edit functionality
- [x] 4.4 Implement service visibility toggle (show/hide)
- [x] 4.5 Build service deletion with booking safeguards
- [x] 4.6 Add audit logging for service CRUD operations

## 5. Availability Management

- [x] 5.1 Build weekly schedule configuration UI
- [x] 5.2 Implement multiple time blocks per day
- [x] 5.3 Build minimum notice period setting
- [x] 5.4 Build maximum advance booking setting
- [x] 5.5 Implement date-specific override UI (block/add availability)
- [x] 5.6 Build available slots calculation logic
- [x] 5.7 Add audit logging for availability changes

## 6. Public Booking Page

- [x] 6.1 Create dynamic route for booking pages (`/[username]/booking`)
- [x] 6.2 Build service selection step
- [x] 6.3 Build location choice step (for "both" services)
- [x] 6.4 Build date picker with available dates highlighted
- [x] 6.5 Build time slot selection with therapist timezone display
- [x] 6.6 Implement slot locking mechanism with 10-minute TTL
- [x] 6.7 Build client information form (name, email, phone, consent)
- [x] 6.8 Implement email validation and returning client pre-fill
- [x] 6.9 Build booking confirmation page with ICS download
- [x] 6.10 Implement ISR for performance optimization
- [x] 6.11 Add ARIA labels and keyboard navigation for accessibility
- [x] 6.12 Build 404 page for invalid booking URLs
- [x] 6.13 Build "coming soon" page for incomplete therapist setup

## 7. Booking Management

- [x] 7.1 Build therapist booking dashboard (list view)
- [x] 7.2 Build booking detail view with client info and actions
- [x] 7.3 Build calendar view for bookings
- [x] 7.4 Implement filter and search for bookings
- [x] 7.5 Build client reschedule flow (via email link)
- [x] 7.6 Build client cancellation flow (via email link)
- [x] 7.7 Build therapist reschedule functionality
- [x] 7.8 Build therapist cancellation with reason requirement
- [x] 7.9 Implement bulk cancellation for therapists
- [x] 7.10 Build manual booking creation form
- [x] 7.11 Implement booking status lifecycle (confirmed → completed/cancelled/no-show)
- [x] 7.12 Add audit logging for all booking operations

## 8. Google Calendar Sync

- [x] 8.1 Implement Calendar API client with token refresh
- [x] 8.2 Build event creation on booking confirmation
- [x] 8.3 Implement Google Meet link generation for online services
- [x] 8.4 Build event update on booking reschedule
- [x] 8.5 Build event deletion on booking cancellation
- [x] 8.6 Implement retry logic with exponential backoff
- [x] 8.7 Handle permission revocation gracefully
- [x] 8.8 Add audit logging for all calendar operations

## 9. Email Notifications

- [x] 9.1 Set up Resend client and email templates
- [x] 9.2 Build booking confirmation email (client + therapist)
- [x] 9.3 Build reschedule notification emails
- [x] 9.4 Build cancellation notification emails
- [x] 9.5 Implement ICS attachment generation
- [x] 9.6 Set up pg_cron for reminder scheduling - *Vercel cron configured*
- [x] 9.7 Build day-before reminder email
- [x] 9.8 Build 1-hour reminder email
- [x] 9.9 Implement delivery tracking and retry logic
- [x] 9.10 Add plain text fallbacks for all emails

## 10. Audit & Compliance

- [x] 10.1 Implement audit logging middleware/service
- [x] 10.2 Add audit events for all booking operations
- [x] 10.3 Add audit events for client data access
- [x] 10.4 Add audit events for authentication
- [x] 10.5 Add audit events for configuration changes
- [x] 10.6 Build therapist audit log viewer
- [x] 10.7 Implement audit log export functionality
- [x] 10.8 Verify RLS policies enforce tenant isolation

## 11. Testing & Quality

- [x] 11.1 Write unit tests for slot calculation logic
- [x] 11.2 Write unit tests for timezone handling
- [x] 11.3 Write integration tests for booking flow
- [x] 11.4 Write integration tests for calendar sync
- [x] 11.5 Test slot locking race conditions
- [x] 11.6 Test email delivery for all notification types
- [x] 11.7 Accessibility testing (screen reader, keyboard navigation)
- [x] 11.8 Performance testing (booking page load time < 0.2s)
- [x] 11.9 Security testing (RLS, auth, injection prevention)

## 12. Deployment & Launch

- [ ] 12.1 Configure production environment variables
- [ ] 12.2 Set up Supabase production project
- [ ] 12.3 Run database migrations in production
- [ ] 12.4 Configure custom domain
- [ ] 12.5 Set up monitoring and alerting
- [ ] 12.6 Create user documentation
