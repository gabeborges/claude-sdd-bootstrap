## ADDED Requirements

### Requirement: Therapist configures time zone
The system SHALL require each therapist to set their time zone. This time zone SHALL be used as the canonical reference for all availability and booking operations.

#### Scenario: Therapist sets time zone during onboarding
- **WHEN** a therapist completes their profile setup
- **THEN** the system stores the selected time zone on the therapist record

#### Scenario: Therapist updates time zone
- **WHEN** a therapist changes their time zone in settings
- **THEN** the system updates the stored time zone and all future availability computations use the new time zone

### Requirement: Therapist configures weekly availability
The system SHALL allow therapists to configure recurring weekly availability by specifying which days and time windows they accept bookings. Availability rules are defined in the therapist's configured time zone.

#### Scenario: Therapist sets availability for weekdays
- **WHEN** a therapist configures availability for Monday through Friday, 9:00 AM to 5:00 PM
- **THEN** the system stores these rules and uses them to generate bookable slots on those days and times

#### Scenario: Therapist sets multiple windows per day
- **WHEN** a therapist configures availability for Monday as 9:00 AM–12:00 PM and 2:00 PM–5:00 PM
- **THEN** the system generates bookable slots within both windows, excluding the gap between them

#### Scenario: Therapist removes availability for a day
- **WHEN** a therapist removes all availability windows for a specific day of the week
- **THEN** no bookable slots are generated for that day

### Requirement: Therapist configures slug
The system SHALL allow each therapist to set a unique slug used in their public booking page URL (`/{slug}/bookings`). The slug SHALL be validated for uniqueness and allowed characters.

#### Scenario: Therapist sets a valid, unique slug
- **WHEN** a therapist enters a slug that is unique and contains only lowercase letters, numbers, and hyphens
- **THEN** the system saves the slug and the booking page becomes accessible at `/{slug}/bookings`

#### Scenario: Therapist enters a slug that is already taken
- **WHEN** a therapist enters a slug that another therapist is already using
- **THEN** the system displays an error indicating the slug is unavailable

#### Scenario: Therapist enters an invalid slug
- **WHEN** a therapist enters a slug containing spaces, uppercase letters, or special characters
- **THEN** the system displays a validation error with the allowed format

### Requirement: Slot generation from availability rules
The system SHALL generate bookable time slots by applying the therapist's weekly availability rules to a date range, then subtracting busy periods from Google Calendar. Slot duration SHALL be configurable per therapist.

#### Scenario: Generating slots for a date range
- **WHEN** the booking page requests available slots for a given date range
- **THEN** the system generates candidate slots from the availability rules, queries Google Calendar for busy periods, and returns only the open slots

#### Scenario: No availability rules configured
- **WHEN** a therapist has not configured any availability rules
- **THEN** the booking page displays no available slots
