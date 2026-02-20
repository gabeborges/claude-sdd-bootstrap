# v0 Task Implementation Order

Recommended implementation sequence across all features, grouped by layer.
Tasks within each layer can be parallelized where dependencies allow.

---

## Layer 1: Database -- Table Creation (DB phases 1-8)

These must be executed in FK dependency order. All are greenfield (no data migration).

| Order | Task ID   | Title                                            | Feature              |
|-------|-----------|--------------------------------------------------|----------------------|
| 1     | T-TS-001  | Create therapists table (phase 1)                | therapist-setup      |
| 2     | T-BS-001  | Create services table (phase 2)                  | booking-services     |
| 3     | T-AV-001  | Create availability table (phase 3)              | availability         |
| 4     | T-BP-002  | Create bookings table (phase 4)                  | booking-page         |
| 5     | T-TS-004  | Create booking_audit_log table (phase 5)         | therapist-setup      |
| 6     | T-BP-001  | Create slot_locks table (phase 6)                | booking-page         |
| 7     | T-GC-001  | Create google_calendar_events table (phase 7)    | google-calendar-sync |
| 8     | T-BN-001  | Create notification_log table (phase 8)          | booking-notifications|

## Layer 2: Database -- RLS Policies (DB phase 9)

Can be applied in parallel after respective tables exist.

| Task ID   | Title                                               | Feature              |
|-----------|-----------------------------------------------------|----------------------|
| T-TS-002  | Apply RLS policies for therapists table              | therapist-setup      |
| T-BS-002  | Apply RLS policies for services table                | booking-services     |
| T-AV-002  | Apply RLS policies for availability table            | availability         |
| T-BP-003  | Apply RLS policies for bookings table                | booking-page         |
| T-BP-004  | Apply RLS policies for slot_locks table              | booking-page         |
| T-GC-002  | Apply RLS policies for google_calendar_events table  | google-calendar-sync |
| T-BN-002  | Apply RLS policies for notification_log table        | booking-notifications|

## Layer 3: Database -- Indexes and Triggers (DB phase 10)

Can be applied in parallel after respective RLS policies exist.

| Task ID   | Title                                               | Feature              |
|-----------|-----------------------------------------------------|----------------------|
| T-TS-003  | Apply indexes/triggers for therapists table          | therapist-setup      |
| T-BS-006  | Apply indexes for services table                     | booking-services     |
| T-AV-007  | Apply indexes for availability table                 | availability         |
| T-BP-005  | Apply indexes for bookings and slot_locks tables     | booking-page         |
| T-GC-003  | Apply indexes for google_calendar_events table       | google-calendar-sync |
| T-BN-003  | Apply indexes for notification_log table             | booking-notifications|

## Layer 4: Auth and Therapist Setup (Backend)

| Order | Task ID   | Title                                            | Depends On                    |
|-------|-----------|--------------------------------------------------|-------------------------------|
| 1     | T-TS-005  | Google OAuth sign-in flow                        | T-TS-001, T-TS-002           |
| 2     | T-TS-006  | Practice details step                            | T-TS-005, T-TS-004           |
| 3     | T-TS-007  | Google Calendar OAuth connection                 | T-TS-006                     |
| 4     | T-TS-009  | Setup resume (persisted state)                   | T-TS-005                     |

## Layer 5: Core Entity CRUD (Backend)

Can be parallelized after Layer 4 completes.

| Task ID   | Title                                               | Depends On                         |
|-----------|-----------------------------------------------------|------------------------------------|
| T-BS-003  | Service CRUD Server Actions                          | T-BS-001, T-BS-002, T-TS-004     |
| T-AV-003  | Availability CRUD Server Actions                     | T-AV-001, T-AV-002, T-TS-004     |
| T-TS-008  | Setup completion logic                               | T-TS-007, T-BS-003, T-AV-003     |

## Layer 6: Booking Flow (Backend)

Depends on services, availability, and therapist setup being in place.

| Order | Task ID   | Title                                            | Depends On                         |
|-------|-----------|--------------------------------------------------|------------------------------------|
| 1     | T-BP-008  | Therapist/service lookup by slug                 | T-BS-001, T-BS-002, T-TS-001     |
| 2     | T-BP-006  | Slot locking mechanism                           | T-BP-001, T-BP-004               |
| 3     | T-AV-004  | Slot computation logic                           | T-AV-003, T-BP-001               |
| 4     | T-BP-007  | Booking creation Server Action                   | T-BP-002, T-BP-003, T-BP-006, T-TS-004 |

## Layer 7: Booking Management (Backend)

Depends on booking creation being in place.

| Order | Task ID   | Title                                            | Depends On                         |
|-------|-----------|--------------------------------------------------|------------------------------------|
| 1     | T-BM-001  | Therapist booking list                           | T-BP-002, T-BP-003               |
| 2     | T-BM-002  | Therapist reschedule                             | T-BP-007, T-BP-006               |
| 3     | T-BM-003  | Therapist cancel                                 | T-BP-002, T-BP-003, T-TS-004    |
| 4     | T-BM-004  | Therapist manual create                          | T-BP-007, T-BP-006               |
| 5     | T-BM-005  | Client access via manage_token                   | T-BP-002, T-BP-003               |
| 6     | T-BM-006  | Client reschedule via token                      | T-BM-005, T-BM-002               |
| 7     | T-BM-007  | Client cancel via token                          | T-BM-005, T-BM-003               |

## Layer 8: Integrations -- Google Calendar Sync

Depends on booking creation and OAuth tokens being in place.

| Order | Task ID   | Title                                            | Depends On                         |
|-------|-----------|--------------------------------------------------|------------------------------------|
| 1     | T-GC-004  | Google Calendar API client                       | T-TS-007                          |
| 2     | T-GC-005  | OAuth token refresh/revocation handling           | T-TS-007, T-GC-004               |
| 3     | T-GC-006  | Calendar sync on booking creation                | T-GC-001-005, T-BP-007, T-TS-004 |
| 4     | T-GC-007  | Calendar sync on reschedule                      | T-GC-006, T-BM-002               |
| 5     | T-GC-008  | Calendar sync on cancellation                    | T-GC-006, T-BM-003               |
| 6     | T-GC-009  | Retry mechanism for failed syncs                 | T-GC-006                          |

## Layer 9: Notifications

Depends on booking lifecycle events and Resend integration.

| Order | Task ID   | Title                                            | Depends On                         |
|-------|-----------|--------------------------------------------------|------------------------------------|
| 1     | T-BN-004  | Configure Resend integration                     | (none)                            |
| 2     | T-BN-005  | Create email templates                           | T-BN-004                          |
| 3     | T-BN-006  | Confirmation email dispatch                      | T-BN-005, T-BP-007, T-BN-001-002 |
| 4     | T-BN-007  | Rescheduling notification dispatch               | T-BN-005, T-BM-002               |
| 5     | T-BN-008  | Cancellation notification dispatch               | T-BN-005, T-BM-003               |
| 6     | T-BN-009  | Reminder scheduling (24h + 1h)                   | T-BN-005, T-BN-001, T-BN-003    |
| 7     | T-BN-010  | Email retry logic                                | T-BN-006                          |

## Layer 10: Frontend -- Therapist Setup

| Order | Task ID   | Title                                            | Depends On                         |
|-------|-----------|--------------------------------------------------|------------------------------------|
| 1     | T-TS-011  | Google OAuth sign-in page                        | T-TS-005                          |
| 2     | T-TS-010  | Setup flow UI (multi-step form)                  | T-TS-006-009                      |

## Layer 11: Frontend -- Core Entity Management

| Task ID   | Title                                               | Depends On                         |
|-----------|-----------------------------------------------------|------------------------------------|
| T-BS-004  | Service management UI                                | T-BS-003                          |
| T-AV-005  | Availability management UI                           | T-AV-003                          |

## Layer 12: Frontend -- Booking Flow

| Order | Task ID   | Title                                            | Depends On                         |
|-------|-----------|--------------------------------------------------|------------------------------------|
| 1     | T-BP-009  | Booking page -- service selection                | T-BP-008                          |
| 2     | T-BP-010  | Booking page -- time slot selection              | T-BP-009, T-AV-004, T-BP-006    |
| 3     | T-BP-011  | Booking page -- client details + confirmation    | T-BP-010, T-BP-007               |
| 4     | T-BP-012  | Booking page -- edge case handling               | T-BP-011                          |

## Layer 13: Frontend -- Booking Management

| Task ID   | Title                                               | Depends On                         |
|-----------|-----------------------------------------------------|------------------------------------|
| T-BM-008  | Therapist booking dashboard UI                       | T-BM-001-004                      |
| T-BM-009  | Client booking management page (manage via token)    | T-BM-005-007                      |

## Layer 14: Testing

All test tasks depend on their respective feature implementations.

| Task ID   | Title                                               | Feature              |
|-----------|-----------------------------------------------------|----------------------|
| T-TS-012  | Therapist setup flow tests                           | therapist-setup      |
| T-BS-005  | Service CRUD tests                                   | booking-services     |
| T-AV-006  | Availability and slot computation tests              | availability         |
| T-BP-013  | Booking page flow tests                              | booking-page         |
| T-BM-010  | Booking management tests                             | booking-management   |
| T-BN-011  | Notification and reminder tests                      | booking-notifications|
| T-GC-010  | Google Calendar sync tests                           | google-calendar-sync |

---

## Cross-Feature Dependencies

Key cross-feature dependency chains:

1. **therapist-setup -> booking-services + availability**: Setup flow delegates to service creation (T-BS-003) and availability setup (T-AV-003) before marking setup complete (T-TS-008).

2. **booking-services + availability -> booking-page**: The booking page requires active services (T-BS-001, T-BS-002) and slot computation from availability (T-AV-004) to function.

3. **booking-page -> booking-management**: Booking management operates on bookings created via the booking page. Shares the booking creation action (T-BP-007) and slot locking mechanism (T-BP-006).

4. **booking-page + booking-management -> google-calendar-sync**: Calendar sync triggers on booking lifecycle events (create from T-BP-007, reschedule from T-BM-002, cancel from T-BM-003).

5. **booking-page + booking-management -> booking-notifications**: Notification dispatch triggers on the same booking lifecycle events as calendar sync.

6. **therapist-setup -> google-calendar-sync**: Calendar sync requires OAuth tokens established during therapist setup (T-TS-007).

---

## Parallelization Opportunities

The following groups can be worked on in parallel by different engineers:

- **Group A (DB)**: All Layer 1-3 tasks (sequential within, but one engineer can do all)
- **Group B (Auth)**: Layer 4 tasks (after therapists table exists)
- **Group C (Services)**: T-BS-003, T-BS-004 (after services table + RLS)
- **Group D (Availability)**: T-AV-003, T-AV-005 (after availability table + RLS)
- **Group E (Notifications)**: T-BN-004, T-BN-005 (Resend setup, email templates -- no DB dependency for config)
- **Group F (GCal client)**: T-GC-004 (after OAuth connection works)

After core backend completes, booking page frontend (Layer 12) and booking management frontend (Layer 13) can be built in parallel.
