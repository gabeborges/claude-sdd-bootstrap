# Capability: Availability

Therapist availability schedule management.

## ADDED Requirements

### Requirement: Weekly Schedule Configuration

The system SHALL allow therapists to set their recurring weekly availability.

#### Scenario: Set availability for a day
- **WHEN** a therapist configures availability for a day of the week
- **THEN** they can specify one or more time blocks (start time, end time)
- **AND** the availability repeats weekly

#### Scenario: Multiple time blocks per day
- **WHEN** a therapist needs a break during the day
- **THEN** they can configure multiple non-overlapping time blocks (e.g., 9am-12pm, 2pm-5pm)

#### Scenario: No availability for a day
- **WHEN** a therapist does not configure any time blocks for a day
- **THEN** that day shows as unavailable on the booking page

#### Scenario: Time block validation
- **WHEN** a therapist configures time blocks
- **THEN** the system prevents overlapping blocks on the same day
- **AND** requires end time to be after start time

### Requirement: Minimum Notice Period

The system SHALL allow therapists to configure minimum booking lead time.

#### Scenario: Configure minimum notice
- **WHEN** a therapist sets a minimum notice period (e.g., 24 hours)
- **THEN** time slots within that window are not available for booking

#### Scenario: Default minimum notice
- **WHEN** a therapist has not configured minimum notice
- **THEN** the system uses a default of 2 hours

#### Scenario: Same-day booking prevention
- **WHEN** a therapist sets minimum notice to 24 hours
- **THEN** clients cannot book appointments for the same day

### Requirement: Maximum Advance Booking

The system SHALL limit how far in advance clients can book.

#### Scenario: Default booking window
- **WHEN** a therapist has not configured maximum advance booking
- **THEN** clients can book up to 60 days in advance

#### Scenario: Custom booking window
- **WHEN** a therapist sets maximum advance booking to 30 days
- **THEN** slots beyond 30 days are not displayed on the booking page

### Requirement: Date-Specific Overrides

The system SHALL allow therapists to override availability for specific dates.

#### Scenario: Block a specific date
- **WHEN** a therapist blocks a specific date (e.g., vacation day)
- **THEN** no time slots are available for that date
- **AND** the weekly schedule is ignored for that date

#### Scenario: Add extra availability
- **WHEN** a therapist adds availability on a normally unavailable day
- **THEN** those time slots become bookable
- **AND** the override takes precedence over weekly schedule

#### Scenario: Modify availability for a date
- **WHEN** a therapist modifies availability for a specific date
- **THEN** the custom hours apply instead of the weekly schedule
- **AND** the change only affects that specific date

### Requirement: Available Slots Calculation

The system SHALL compute available booking slots based on services and availability.

#### Scenario: Slot generation
- **WHEN** a client views the booking page
- **THEN** available slots are calculated from availability minus existing bookings
- **AND** slots are sized according to the selected service duration

#### Scenario: Slot alignment
- **WHEN** generating available slots
- **THEN** slots start at consistent intervals (e.g., on the hour or half-hour)
- **AND** slot duration matches the selected service

#### Scenario: Existing bookings block slots
- **WHEN** a time slot overlaps with an existing booking
- **THEN** that slot is not available for new bookings
