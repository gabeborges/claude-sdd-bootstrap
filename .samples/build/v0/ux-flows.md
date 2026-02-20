# UX Flows -- Curio Booking & Scheduling (v0)

**Design system reference:** `.ops/ui-design-system.md`
**Palette constraint:** No purple/blue. Sage green accent (`--sage: #5B7A5E`).

---

## 1. Therapist Dashboard (Shell & Navigation)

### Entry Point
Therapist signs in via Google OAuth -> lands on dashboard home.
Subsequent visits: direct to dashboard home (authenticated).

### Layout
```
[Sidebar 240px] | [Main Content Area]
                  [Page Header: title + primary action]
                  [Content: max 960px for forms, full-width for schedule views]
```

### Sidebar Navigation
- **Items:** Dashboard (home icon), Services, Availability, Bookings
- **States per item:**
  - Default: `--ink-secondary`, `--radius-sm`
  - Hover: `--surface-inset` background
  - Active (current page): `--sage-subtle` background, `--sage` text, weight 500
  - Focus: `--control-focus-ring`
- **Footer area:** Therapist name (from Google profile), Settings link, Sign out
- **Responsive:** Sidebar collapses to hamburger menu on tablet/mobile (breakpoint: 768px)
- **Accessibility:**
  - `<nav aria-label="Main navigation">`
  - Nav items are `<a>` or `<button>` with `aria-current="page"` on active item
  - Keyboard: Tab through items, Enter to activate
  - Skip link: "Skip to main content" as first focusable element

### Screens

#### Dashboard Home
- **States:**
  - Loading: Skeleton cards (3 session card skeletons) + skeleton stat blocks
  - Empty (no bookings): "No upcoming sessions" message with CTA "Set up your services" or "Share your booking page"
  - Populated: Upcoming sessions (next 7 days) as session cards + quick stats
  - Error: Retry banner at top of content area, `--destructive-subtle` background
- **Key elements:**
  - **Quick stats row:** Today's sessions count, This week total, Next available slot
    - Each stat: label (`--ink-tertiary`, text-sm, uppercase) + value (`--ink`, text-xl, tabular-nums)
  - **Upcoming sessions list:** Session cards (signature element)
  - **Booking page link:** Copy-to-clipboard of `curio.health/{slug}/booking` with confirmation toast
- **Session card structure (per design system):**
  - Left: Time block (monospace, `--ink`, weight 500)
  - Center: Client initials circle (`--sage-subtle` bg, `--sage` text) + client name (`--ink`, weight 500) + service name (`--ink-tertiary`, text-sm)
  - Right: Status badge + duration
  - Border-left: 3px solid `--sage` (confirmed) / `--warning` (pending) / `--border` (past)
  - Surface: `--surface-1`, border: `--border-subtle`, shadow: `--shadow`, padding: `--space-4`, radius: `--radius`
- **Accessibility:**
  - Session list: `<ul>` with `aria-label="Upcoming sessions"`
  - Each session card: `<li>` with meaningful content order for screen readers (date, time, client name, service, status)
  - Stats use `aria-label` on each value (e.g., `aria-label="3 sessions today"`)

---

## 2. Therapist Setup Flow (Onboarding Wizard)

### Entry Point
New therapist completes Google OAuth -> redirected to setup wizard.
Returning therapist with incomplete setup -> resumes at last completed step.

### Progress Indicator
- Horizontal step bar at top of setup content area
- Steps: 1. Practice Details | 2. Google Calendar | 3. First Service | 4. Availability | 5. Done
- Step states: completed (checkmark, `--sage`), current (filled circle, `--sage`), upcoming (empty circle, `--ink-muted`)
- `aria-label="Setup progress: Step {n} of 5, {step name}"`
- `role="progressbar"` with `aria-valuenow` and `aria-valuemax`

### Screens

#### Step 1: Practice Details
- **States:**
  - Default: Form with pre-filled name from Google profile
  - Validating: Inline errors on blur
  - Submitting: "Continue" button shows loading spinner, form disabled
  - Error: Inline field errors + banner for server errors
- **Key elements:**
  - Practice name input (pre-filled from Google profile name, editable)
    - Validation: Required, max 100 characters
  - Timezone select dropdown
    - Default: Auto-detected from browser, user can change
    - Options: IANA timezone list, grouped by region
    - Validation: Required
  - Booking URL slug input
    - Auto-generated from practice name (kebab-case), editable
    - Real-time uniqueness check (debounced 500ms)
    - Validation: Required, unique, alphanumeric + hyphens, 3-50 chars
    - Preview: "Your booking page: curio.health/{slug}/booking"
  - "Continue" primary button (`--sage` bg)
- **Accessibility:**
  - All inputs: `<label>` associated via `for` attribute
  - Required fields: `aria-required="true"`
  - Inline errors: `aria-describedby` linking input to error message
  - Timezone dropdown: Headless UI Listbox with keyboard navigation (arrow keys, type-ahead)
  - Slug uniqueness: `aria-live="polite"` region announces availability result

#### Step 2: Connect Google Calendar
- **States:**
  - Default: Explanation text + "Connect Google Calendar" button
  - Connecting: Button disabled, loading indicator, "Connecting..." text
  - Success: Checkmark + "Google Calendar connected" confirmation, auto-advance or "Continue" button
  - Error: Error message explaining what went wrong + "Try again" button
  - Denied: Specific message "Calendar access is required for booking to function" + "Try again"
- **Key elements:**
  - Explanation: Why Calendar access is needed (one-way sync, Meet links for online sessions)
  - "Connect Google Calendar" button (`--sage` bg, Google icon)
  - Permission scope disclosure: "Curio will only create events on your calendar. It will not read or delete existing events."
- **Accessibility:**
  - Focus returns to "Try again" button after OAuth redirect failure
  - `aria-live="assertive"` for connection status announcements
  - Error messages linked to actionable element

#### Step 3: Create First Service
- **States:**
  - Default: Service creation form (same as Services Management create form)
  - Validating: Inline errors on blur
  - Submitting: "Create & Continue" button loading
  - Success: Brief confirmation, auto-advance to Step 4
  - Error: Inline field errors + server error banner
- **Key elements:**
  - Service name input (required, max 100 chars)
  - Duration select (dropdown: 15, 30, 45, 60, 75, 90, 120 min)
  - Modality toggle: Online / In-Person (radio group)
    - Online: Note "Google Meet link will be generated automatically"
    - In-Person: Note "No video link will be created"
  - Description textarea (optional, max 500 chars)
  - "Create & Continue" primary button
  - "Skip for now" ghost link (disabled -- at least one service required per spec R6)
- **Accessibility:**
  - Modality: `<fieldset>` with `<legend>"Session type"</legend>`, radio inputs
  - Duration: Headless UI Listbox
  - Character count on description: `aria-live="polite"` updates

#### Step 4: Set Availability
- **States:**
  - Default: Weekly grid with no blocks set
  - Editing: Active day expanded with time block inputs
  - Validating: Overlap errors shown inline
  - Submitting: "Complete Setup" button loading
  - Error: Overlap error per-block or server error banner
- **Key elements:**
  - Day selector: 7-day row (Sun-Sat), toggle each day on/off
    - Active day: `--sage-subtle` bg, `--sage` border
    - Inactive day: `--surface-inset` bg, `--ink-muted` text
  - Per active day: Time block list
    - Each block: Start time + End time (time select dropdowns, 15-min increments)
    - "Add time block" ghost button per day
    - Remove block: icon button with `aria-label="Remove time block"`
  - Minimum notice input: Number input + "hours" label (default: 24)
  - Timezone display: Read-only, set in Step 1 (e.g., "America/Toronto (ET)")
  - "Complete Setup" primary button
- **Accessibility:**
  - Day toggles: `role="checkbox"` or toggle buttons with `aria-pressed`
  - Time selects: Headless UI Listbox, arrow keys navigate
  - Time block list per day: `role="list"`, remove button announces "Remove 9:00 AM to 12:00 PM block"
  - Overlap error: `aria-live="assertive"`, `aria-describedby` on conflicting block

#### Step 5: Setup Complete
- **States:**
  - Populated only (always shown after successful setup)
- **Key elements:**
  - Success message: "Your booking page is live"
  - Booking page URL: `curio.health/{slug}/booking` with copy button
  - "Preview Booking Page" link (opens in new tab)
  - "Go to Dashboard" primary button
- **Accessibility:**
  - Focus auto-moves to success heading on arrival
  - Copy button: `aria-label="Copy booking page URL"`, announces "Copied" via `aria-live="polite"`

### Transitions
```
Google OAuth -> Step 1: Practice Details
Step 1 "Continue" -> Step 2: Connect Google Calendar
Step 2 success -> Step 3: Create First Service
Step 3 "Create & Continue" -> Step 4: Set Availability
Step 4 "Complete Setup" -> Step 5: Setup Complete
Step 5 "Go to Dashboard" -> Dashboard Home
```

### Error Handling
- OAuth failure: Redirect to sign-in page with error message
- Any step server error: Inline error banner with retry; form remains populated
- Network error at any step: "Unable to save. Check your connection and try again."
- Abandoned setup: Progress persisted via `setup_step` field; resume on next sign-in

---

## 3. Services Management

### Entry Point
Sidebar -> "Services"

### Screens

#### Service List
- **States:**
  - Loading: 3 skeleton cards matching session card layout
  - Empty: "No services yet" + "Create your first service" CTA button (centered, `--ink-tertiary` text)
  - Populated: List of service cards
  - Error: Retry banner at top, `--destructive-subtle` bg
- **Key elements:**
  - Page header: "Services" (text-xl) + "Add Service" primary button
  - Service card (per service):
    - Service name (`--ink`, weight 500)
    - Duration badge (text-data, e.g., "60 min")
    - Modality badge: "Online" (`--sage-subtle` bg) / "In-Person" (`--surface-inset` bg)
    - Description preview (1 line, truncated, `--ink-tertiary`)
    - Status: Active/Inactive toggle
    - Actions: Edit (ghost button), Delete (ghost button, `--destructive` on hover)
    - Surface: `--surface-1`, border: `--border-subtle`, shadow: `--shadow`, radius: `--radius`
  - Active/Inactive toggle:
    - Toggle switch; active = `--sage`, inactive = `--ink-muted`
    - Toggling to inactive removes service from public booking page
    - Toggling emits confirmation toast: "Service deactivated" / "Service activated"
- **Accessibility:**
  - Service list: `<ul aria-label="Your services">`
  - Each card: `<li>` with service name as accessible name
  - Toggle: `role="switch"` with `aria-checked` and `aria-label="Active status for {service name}"`
  - Edit/Delete: `aria-label="Edit {service name}"` / `aria-label="Delete {service name}"`
  - Keyboard: Tab through cards, Tab to actions within card

#### Create/Edit Service (Modal or Inline Panel)
- **States:**
  - Default: Form with empty fields (create) or pre-filled fields (edit)
  - Validating: Inline errors on blur
  - Submitting: Save button loading, form disabled
  - Success: Modal closes, list refreshes, toast confirmation
  - Error: Inline field errors + server error banner in modal
- **Key elements:**
  - Service name input (required, max 100 chars)
  - Duration select (15, 30, 45, 60, 75, 90, 120 minutes)
  - Modality radio group: Online / In-Person
  - Description textarea (optional, max 500 chars, character counter)
  - "Save" primary button / "Cancel" secondary button
  - Edit mode: Title says "Edit Service", pre-filled values
  - Create mode: Title says "New Service", empty form
- **Accessibility:**
  - Modal: `role="dialog"`, `aria-modal="true"`, `aria-labelledby` to title
  - Focus trap: Tab cycles within modal
  - Escape closes modal, focus returns to trigger button
  - All fields: `<label>` with `for`, `aria-required` on required fields
  - Errors: `aria-describedby` linking field to error text

#### Delete Confirmation (Dialog)
- **States:**
  - Default: Confirmation prompt
  - Has bookings warning: Additional warning text about existing bookings
  - Submitting: "Delete" button loading
  - Error: Error message within dialog
- **Key elements:**
  - Title: "Delete {service name}?"
  - Body: "This service will be removed from your booking page. Existing bookings will not be affected."
  - If service has bookings: Warning callout `--warning-subtle` bg: "{n} existing bookings will remain on your schedule."
  - "Delete" destructive button / "Cancel" secondary button
- **Accessibility:**
  - `role="alertdialog"`, `aria-modal="true"`
  - Focus moves to "Cancel" button on open (safe default)
  - Escape closes without deleting

### Transitions
```
Service List -> "Add Service" -> Create Service Modal
Service List -> "Edit" on card -> Edit Service Modal
Service List -> "Delete" on card -> Delete Confirmation Dialog
Create/Edit Modal -> "Save" success -> Service List (refreshed)
Delete Dialog -> "Delete" success -> Service List (refreshed, toast: "Service deleted")
```

### Error Handling
- List load failure: Retry banner; no stale data shown
- Create/Edit server error: Error message in modal; form stays populated for retry
- Delete failure: Error message in dialog; service remains
- Duplicate name: Not enforced in v0 (no constraint in spec)

---

## 4. Availability Management

### Entry Point
Sidebar -> "Availability"

### Screens

#### Weekly Availability View
- **States:**
  - Loading: 7-day skeleton grid with placeholder blocks
  - Empty: All days shown as inactive; "Set your first available hours" prompt with CTA
  - Populated: Days with time blocks displayed
  - Error: Retry banner at top
- **Key elements:**
  - Page header: "Availability" (text-xl)
  - Timezone display: Read-only badge showing current timezone (e.g., "America/Toronto (ET)"), with "Change" link to settings
  - Minimum notice input: Number field + "hours minimum notice" label, inline save (auto-save on blur or explicit save button)
  - Weekly grid layout:
    - 7 columns (or 7 rows on mobile), one per day
    - Column header: Day name (Mon, Tue, etc.)
    - Day toggle: Clickable to enable/disable day
    - Active day: Time blocks displayed as vertical bars
      - Block visual: `--sage-subtle` bg, `--sage` left border (3px), start/end times as text-data
      - Hover: `--border-strong` border, edit affordance
    - Inactive day: `--surface-inset` bg, `--ink-muted` "Not available" text
  - "Add time block" button per active day (appears on hover/focus or always visible)
  - Edit block: Click block to open edit popover (start time, end time, remove)
  - Remove block: Trash icon within edit popover
  - Save behavior: Auto-save with debounce on each change, or explicit "Save changes" button with dirty-state indicator
- **Accessibility:**
  - Grid: `role="grid"` with `aria-label="Weekly availability schedule"`
  - Day columns: `role="columnheader"` for day names
  - Day toggle: `role="switch"` with `aria-checked`, `aria-label="Available on {day}"`
  - Time blocks: Focusable, `aria-label="{day} {start} to {end}"`, Enter to edit
  - Add block: `aria-label="Add time block on {day}"`
  - Keyboard: Arrow keys navigate between days, Tab into blocks within a day
  - Overlap error: `aria-live="assertive"` announcement + inline error on conflicting block
  - Minimum notice: `<label>` associated, `aria-describedby` for help text

#### Edit Time Block (Popover)
- **States:**
  - Default: Start/end time selects pre-filled
  - Validating: Overlap warning shown
  - Saving: Brief loading state
  - Error: Inline error for overlap or invalid range
- **Key elements:**
  - Start time select (15-min increments, 6:00 AM - 11:45 PM)
  - End time select (must be after start, 15-min increments)
  - "Remove" destructive ghost button
  - Auto-close on click outside or Escape
- **Accessibility:**
  - Popover: `role="dialog"`, `aria-label="Edit time block"`
  - Focus trap within popover
  - Escape closes, focus returns to block that opened it
  - Time selects: Headless UI Listbox

### Transitions
```
Availability View -> click day toggle -> enable/disable day
Availability View -> click "Add time block" on day -> Add Block Popover
Availability View -> click existing block -> Edit Block Popover
Edit Popover -> "Remove" -> block removed, popover closes
Edit Popover -> change times -> inline save, popover closes
Availability View -> change minimum notice -> inline save
```

### Error Handling
- Load failure: Retry banner
- Save failure (overlap): Inline error on conflicting block, clear error message: "This block overlaps with {existing block time}. Adjust the times."
- Save failure (server): Toast error "Failed to save. Try again."
- Invalid time range (end <= start): Inline error "End time must be after start time"

### Responsive
- Desktop: 7-column grid
- Tablet: 7-column compressed or horizontal scroll
- Mobile: Stacked list view, one day per row, expandable to show/edit blocks

---

## 5. Public Booking Page (Client-Facing)

### Entry Point
Client navigates to `curio.health/{slug}/booking` (public URL, no auth required).

### Layout
- No sidebar. Clean, single-column layout.
- Max-width: 640px, centered.
- Mobile-first responsive design.
- Minimal JS for performance target (~0.2s load).

### Screens

#### Service Selection
- **States:**
  - Loading: 2-3 skeleton cards
  - Empty (no active services): "Online booking is not yet available. Please contact {practice name} directly." No error styling.
  - Populated: Service cards for selection
  - Error: "Unable to load booking page. Please try again." with retry link
- **Key elements:**
  - Page header: "{Practice Name}" (text-xl), "Book a Session" subtitle
  - Timezone notice: "All times shown in {timezone label}" (`--ink-tertiary`, text-sm)
  - Service cards (one per active service):
    - Service name (`--ink`, weight 500)
    - Duration (`--ink-tertiary`, text-data)
    - Modality icon + label: "Video Session" (online) / "In-Person" (in-person)
    - Description (if present, `--ink-secondary`, text-sm)
    - Entire card is clickable; hover: `--border-strong` border
    - Surface: `--surface-1`, border: `--border-subtle`, shadow: `--shadow`, radius: `--radius`
  - No therapist email, phone, or internal IDs exposed (data minimization)
- **Accessibility:**
  - Service cards: `role="radio"` within `role="radiogroup"` with `aria-label="Select a service"`, or each card as a link/button
  - Keyboard: Tab between services, Enter/Space to select
  - Focus indicator: `--control-focus-ring` on card
  - Screen reader: Announces service name, duration, modality on focus

#### Date & Time Selection
- **States:**
  - Loading: Calendar skeleton + "Loading available times..." text
  - Empty (no slots available): "No available times for this service. Please check back later or contact {practice name}."
  - Populated: Calendar with available dates + time slots for selected date
  - Error: "Unable to load available times. Please try again." with retry
  - Slot taken (race condition): "This time is no longer available. Please select another time." with `--warning-subtle` bg
- **Key elements:**
  - Back link: "Back to services" (ghost button)
  - Selected service summary: Name + duration badge at top
  - Date picker: Month calendar grid
    - Available dates: Default style, clickable
    - Unavailable dates: `--ink-muted`, not clickable, `aria-disabled="true"`
    - Selected date: `--sage` bg, white text
    - Today: Subtle ring indicator
    - Navigation: Previous/next month arrows
  - Time slot grid (for selected date):
    - Available slots: Buttons in a grid/list, `--surface-1` bg, `--border`
    - Hover: `--sage-ghost` bg, `--border-strong`
    - Selected: `--sage` bg, white text
    - Time format: "9:00 AM", "9:30 AM" etc. (monospace/tabular-nums)
  - Slot lock visual: After selecting a time, brief "Reserving your time..." indicator (300ms), then transition to booking form
    - If lock fails: "This time was just taken. Please select another." warning
- **Accessibility:**
  - Calendar: `role="grid"` with `aria-label="Select a date"`, `aria-roledescription="calendar"`
  - Date cells: `role="gridcell"`, `aria-selected` on chosen date, `aria-disabled` on unavailable
  - Keyboard: Arrow keys navigate dates, Enter selects; Tab moves to time slots
  - Month nav: Previous/Next as buttons with `aria-label="Previous month"` / `aria-label="Next month"`
  - Time slots: `role="radiogroup"` with `aria-label="Available times"`, each slot `role="radio"`
  - Slot lock status: `aria-live="polite"` region announces reservation confirmation or failure
  - Focus management: After date select, focus moves to first available time slot

#### Booking Details Form
- **States:**
  - Default: Empty form with slot summary at top
  - Validating: Inline errors on blur
  - Submitting: "Confirm Booking" button loading, form disabled
  - Slot expired (lock timeout): "Your reserved time has expired. Please go back and select a new time." with back button
  - Error (server): Error banner + form remains populated for retry
  - Error (slot taken on submit): "This time is no longer available." with back-to-time-selection link
- **Key elements:**
  - Booking summary card: Service name, date, time, duration (read-only, `--surface-inset` bg)
  - Full name input (required)
  - Email input (required, format validation)
  - Phone input (required, format validation: Canadian 10-digit or international with country code)
  - Consent checkbox (required, unchecked by default):
    - Label: "I consent to {Practice Name} collecting and storing my name, email, and phone number for the purpose of scheduling and managing my appointment."
    - Must be explicit affirmative action
  - "Confirm Booking" primary button (`--sage` bg)
  - "Back" ghost link to return to time selection (releases slot lock)
  - Slot hold timer: Subtle countdown or "Your time is held for {n} minutes" text (`--ink-tertiary`)
- **Accessibility:**
  - All inputs: `<label>` with `for`, `aria-required="true"` on required fields
  - Inline errors: `role="alert"` or `aria-describedby` linking field to error
  - Consent: `<input type="checkbox">` with full label text (not truncated), `aria-required="true"`
  - Phone: `aria-describedby` to format hint
  - Submit button disabled state: `aria-disabled` (not removed from DOM)
  - Focus on first error field after failed submission

#### Booking Confirmation
- **States:**
  - Populated only (always shown after successful booking)
- **Key elements:**
  - Success icon/indicator: Checkmark with `--sage` color
  - "Booking Confirmed" heading
  - Booking details card:
    - Service name
    - Date and time (in therapist timezone with label)
    - Duration
    - Modality: "Video Session" (with note: "A Google Meet link will be sent to your email") or "In-Person"
  - "A confirmation email has been sent to {email}." message
  - "You can manage your booking from the link in your confirmation email."
  - No further actions needed; page is a terminal state
- **Accessibility:**
  - Focus moves to "Booking Confirmed" heading on arrival (`tabindex="-1"`, auto-focus)
  - `role="status"` on confirmation container
  - All booking details accessible to screen readers

### Transitions
```
Service Selection -> click service card -> Date & Time Selection
Date & Time Selection -> "Back to services" -> Service Selection
Date & Time Selection -> select date -> show time slots for that date
Date & Time Selection -> select time slot -> lock slot -> Booking Details Form
Booking Details Form -> "Confirm Booking" success -> Booking Confirmation
Booking Details Form -> "Back" -> Date & Time Selection (slot released)
Booking Details Form -> slot expired -> Slot Expired state with back link
```

### Error Handling
- Page load failure: Full-page error with retry link
- No services: Friendly "not yet available" message (not error styling)
- No slots: Friendly "no times available" message
- Slot lock failure (race condition): Warning message, return to time selection
- Slot expired during form: Warning message with back link to re-select
- Form validation failure: Inline errors, focus first error field
- Submit failure (server): Error banner, form stays populated
- Submit failure (double-book): Specific "time no longer available" message, link to time selection

### Responsive
- Mobile (< 640px): Full-width, stacked layout. Calendar fills width. Time slots in 2-column grid.
- Tablet (640px-1024px): Centered 640px column.
- Desktop (> 1024px): Centered 640px column.
- Touch targets: Minimum 44x44px on all interactive elements (WCAG 2.5.5).

### Performance
- Server-rendered service list (Server Component)
- Date/time picker: Client component, loaded after initial paint
- No unnecessary JS bundles; tree-shake aggressively
- Slot data fetched on date selection (not full month prefetch)

---

## 6. Booking Management (Therapist View)

### Entry Point
Sidebar -> "Bookings"

### Screens

#### Booking List
- **States:**
  - Loading: Skeleton session cards (5 rows)
  - Empty: "No bookings yet. Share your booking page to start receiving appointments." with copy-link CTA
  - Populated: Session cards grouped by date
  - Error: Retry banner
- **Key elements:**
  - Page header: "Bookings" (text-xl) + "Create Booking" primary button
  - Filter tabs: Upcoming (default) | Past | Cancelled
    - Active tab: `--sage` underline, `--ink` text
    - Inactive tab: `--ink-tertiary` text, hover: `--ink-secondary`
  - Session cards (per booking, using signature session card pattern):
    - Left: Date + time block (monospace)
    - Center: Client initials circle + client name + service name
    - Right: Status badge (Confirmed / Cancelled / Completed) + duration
    - Border-left: 3px solid `--sage` (confirmed) / `--destructive` (cancelled) / `--border` (past/completed)
    - Click to expand or navigate to booking detail actions
  - Expanded card actions (or inline): Reschedule (ghost button), Cancel (ghost destructive button)
  - Date group headers: "Today", "Tomorrow", "Wed, Feb 12" etc. (`--ink-tertiary`, text-sm, uppercase)
- **Accessibility:**
  - Filter tabs: `role="tablist"` + `role="tab"` + `role="tabpanel"` with `aria-selected`
  - Keyboard: Arrow keys between tabs, Tab into card list
  - Card list: `<ul aria-label="Bookings">`, each card `<li>`
  - Card actions: Accessible via Tab within focused card, `aria-label="Reschedule booking with {client name}"` etc.
  - Status badges: Color + text (never color alone)

#### Create Booking (Modal)
- **States:**
  - Default: Multi-step form (select service -> select slot -> enter client details)
  - Service loading: Skeleton options
  - Slots loading: Skeleton time grid
  - Submitting: "Create" button loading
  - Success: Modal closes, list refreshes, toast "Booking created"
  - Error: Inline errors + server error banner
- **Key elements:**
  - Step 1: Select service (radio list of therapist's active services)
  - Step 2: Select date & time (same date picker + time slots as booking page)
  - Step 3: Client details (name, email, phone, consent checkbox -- therapist confirms on behalf)
  - "Create Booking" primary button / "Cancel" secondary
  - Progress within modal: Step indicator or breadcrumb
- **Accessibility:**
  - Modal: `role="dialog"`, `aria-modal="true"`, focus trap
  - Same form accessibility as booking page form
  - Escape closes modal at any step

#### Reschedule Flow (Modal)
- **States:**
  - Default: Current booking summary + date/time picker for new slot
  - Slots loading: Skeleton
  - Submitting: "Confirm Reschedule" button loading
  - Success: Modal closes, list refreshes, toast "Booking rescheduled"
  - Error (slot unavailable): "This time is not available" inline error
  - Error (server): Server error banner
- **Key elements:**
  - Current booking info: Service, client name, original date/time (read-only, `--surface-inset` bg)
  - Date picker + time slot selection (same pattern as booking page)
  - "Confirm Reschedule" primary button / "Cancel" secondary
- **Accessibility:**
  - Same calendar/time picker accessibility as booking page
  - Focus management: Return to trigger card on close

#### Cancel Booking (Dialog)
- **States:**
  - Default: Confirmation prompt
  - Submitting: "Cancel Booking" button loading
  - Success: Dialog closes, list refreshes, toast "Booking cancelled"
  - Error: Error message in dialog
- **Key elements:**
  - Title: "Cancel booking with {client name}?"
  - Booking summary: Service, date, time
  - Cancellation reason textarea (optional)
  - "Cancel Booking" destructive button / "Keep Booking" secondary button
- **Accessibility:**
  - `role="alertdialog"`, `aria-modal="true"`
  - Focus moves to "Keep Booking" (safe default) on open
  - Escape closes without cancelling

### Transitions
```
Booking List -> "Create Booking" -> Create Booking Modal
Booking List -> "Reschedule" on card -> Reschedule Modal
Booking List -> "Cancel" on card -> Cancel Confirmation Dialog
Reschedule Modal -> "Confirm" success -> Booking List (refreshed)
Cancel Dialog -> "Cancel Booking" success -> Booking List (refreshed)
Create Modal -> "Create" success -> Booking List (refreshed)
```

### Error Handling
- List load failure: Retry banner
- Create/Reschedule slot conflict: Inline slot-unavailable error, pick different time
- Cancel failure: Error message in dialog, booking unchanged
- Network error on any action: Toast error with retry guidance

---

## 7. Client Booking Management (via Tokenized Link)

### Entry Point
Client clicks manage-booking link from confirmation/notification email.
URL format: `curio.health/manage/{manage_token}`

### Layout
- Same as public booking page: Single-column, max-width 640px, centered. No sidebar, no auth.

### Screens

#### Booking Detail (Client View)
- **States:**
  - Loading: Skeleton card
  - Populated: Booking details with management actions
  - Invalid/expired token: "This link is no longer valid or has expired. Please check your email for an updated link, or contact your therapist."
  - Already cancelled: Booking details shown (read-only) with "Cancelled" status badge; no actions
  - Already completed: Booking details shown (read-only) with "Completed" status; no actions
  - Error: "Unable to load your booking. Please try again." with retry
- **Key elements:**
  - Booking details card:
    - Service name
    - Date and time (therapist timezone with label)
    - Duration
    - Modality + Meet link (if online, shown only after confirmation email)
    - Status badge
  - Actions (for confirmed bookings only):
    - "Reschedule" secondary button
    - "Cancel Booking" ghost destructive button
- **Accessibility:**
  - Booking details: Semantic headings and description list
  - Buttons: `aria-label` with context ("Reschedule your {service name} appointment")
  - Token error: `role="alert"`

#### Client Reschedule Flow
- **States:**
  - Default: Date/time picker (same as booking page)
  - Loading slots: Skeleton
  - Submitting: "Confirm New Time" button loading
  - Success: Confirmation message with updated details
  - Slot unavailable: Warning + pick different time
  - Error: Server error banner
- **Key elements:**
  - Current booking summary (read-only)
  - Date picker + time slots (same patterns as booking page)
  - "Confirm New Time" primary button / "Go Back" ghost button
- **Accessibility:**
  - Same as booking page date/time picker accessibility
  - Success: Focus moves to confirmation heading

#### Client Cancel Flow (Inline Confirmation)
- **States:**
  - Default: Confirmation prompt (inline, not modal -- single page context)
  - Submitting: "Confirm Cancellation" button loading
  - Success: Updated booking detail showing "Cancelled" status
  - Error: Error message with retry
- **Key elements:**
  - Confirmation text: "Are you sure you want to cancel your {service name} appointment on {date} at {time}?"
  - "Confirm Cancellation" destructive button / "Go Back" secondary button
- **Accessibility:**
  - `aria-live="assertive"` for cancellation result
  - Focus moves to result heading after action

### Transitions
```
Email link -> Booking Detail (Client View)
Booking Detail -> "Reschedule" -> Client Reschedule Flow
Booking Detail -> "Cancel Booking" -> Client Cancel Confirmation
Client Reschedule -> "Confirm" success -> Booking Detail (updated)
Client Cancel -> "Confirm" success -> Booking Detail (shows "Cancelled")
Client Reschedule -> "Go Back" -> Booking Detail
Client Cancel -> "Go Back" -> Booking Detail
```

### Error Handling
- Invalid token: Friendly "link no longer valid" message (not technical error)
- Expired token: Same as invalid token message
- Reschedule slot conflict: Warning, pick different time
- Cancel failure: Error message, booking unchanged
- Network error: "Unable to process. Please try again."

### Responsive
- Same as public booking page: Mobile-first, 640px max-width, centered.

---

## Design System Compliance Checklist

All screens MUST comply with:

| Requirement | Standard |
|---|---|
| Focus indicators | `--control-focus-ring` on all interactive elements |
| Color independence | Status always communicated via text + color (never color alone) |
| Text contrast | 4.5:1 minimum (normal text), 3:1 (large text, UI components) |
| Touch targets | 44x44px minimum on mobile (WCAG 2.5.5) |
| Form labels | Every input has associated `<label>` |
| Error association | `aria-describedby` linking fields to error messages |
| Keyboard operability | All flows completable via keyboard only |
| Screen reader | Meaningful announcements for all state changes via `aria-live` |
| Animation | Respect `prefers-reduced-motion` media query |
| Font | Inter for body, monospace for times/data |
| Depth | Cards: `--shadow` + `--border-subtle`; Modals: `--shadow-lg`; Popovers: `--shadow-md` |

---

## Design Decisions Made (for Open Questions)

The following UX decisions are recommended to resolve open spec questions:

| Question | Decision | Rationale |
|---|---|---|
| Services Q1: Active/inactive toggle? | Yes, include toggle | Allows temporary hiding without data loss; better UX than delete-only |
| Services Q2: Duration bounds | 15 min minimum, 120 min maximum | Covers standard therapy session lengths |
| Services Q3: Max lengths | Name: 100 chars, Description: 500 chars | Reasonable for display without truncation issues |
| Availability Q1: Time granularity | 15-minute increments | Standard for therapy scheduling; balances flexibility and simplicity |
| Availability Q2: Future availability window | 8 weeks rolling | Practical booking horizon; prevents unbounded queries |
| Availability Q3: Show timezone to client? | Yes, always shown | "All times shown in {timezone}" prevents confusion |
| Booking Page Q2: Slot lock timeout | 10 minutes | Enough time to fill form; not so long it blocks other clients |
| Booking Page Q3: Phone format | Accept both Canadian (10-digit) and international (E.164) | Flexibility for diverse client base |
| Booking Page Q5: Timezone label | Yes (see Availability Q3) | Critical for client understanding |
| Booking Mgmt Q4: Cancellation reason | Optional free-text (therapist only) | Low friction; data capture without mandate |
| Therapist Setup Q1: Slug source | Auto-generated from practice name, editable | User control with sensible default |
