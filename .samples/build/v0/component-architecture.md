# Component Architecture -- Curio Booking & Scheduling (v0)

**Source**: `.ops/build/v0/ux-flows.md`
**Design system**: `.ops/ui-design-system.md`
**Data model**: `.ops/build/system-design.yaml`

---

## 1. Shared/Base Components

These are the foundational components reused across all features. This is a greenfield project -- all components are new.

---

### Component: PageLayout

**Purpose**: Authenticated therapist shell with sidebar navigation and main content area.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| title | string | yes | Page header title text (text-xl) |
| action | ReactNode | no | Primary action element rendered in page header (e.g., "Add Service" button) |
| fullWidth | boolean | no | When true, content area has no max-width constraint (for calendar/schedule views). Default false (max 960px). |
| children | ReactNode | yes | Main content area |

#### States
- **Default**: Sidebar visible (240px), main content rendered.
- **Mobile (< 768px)**: Sidebar collapsed behind hamburger menu overlay.
- **Loading**: Layout shell renders immediately; children control their own loading states.

#### Children
- `Sidebar` -- Navigation panel (always rendered within layout)
- `SkipLink` -- "Skip to main content" as first focusable element
- `PageHeader` -- Title + action slot

#### Accessibility
- `<main>` wraps content area with `id="main-content"` for skip link target.
- Sidebar renders inside `<nav aria-label="Main navigation">`.
- Hamburger menu toggle: `aria-expanded`, `aria-controls`.

#### Data Requirements
- Therapist profile (name for sidebar footer): read from auth session or server component prop.

---

### Component: Sidebar

**Purpose**: Left navigation panel for authenticated therapist routes.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| currentPath | string | yes | Current route path for active state |
| therapistName | string | yes | Display name for footer |
| onSignOut | () => void | yes | Sign out handler |

#### States
- **Default (desktop)**: Visible, 240px fixed.
- **Collapsed (mobile)**: Hidden, toggleable via hamburger.
- **Open (mobile)**: Overlay drawer with backdrop.

#### Children
- `NavItem` -- Repeated for each route (Dashboard, Services, Availability, Bookings)
- `SidebarFooter` -- Therapist name, Settings link, Sign out

#### Accessibility
- `<nav aria-label="Main navigation">`
- Active item: `aria-current="page"`
- Mobile overlay: focus trap, Escape to close, `aria-modal="true"` on drawer

---

### Component: NavItem

**Purpose**: Single navigation link within the sidebar.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| href | string | yes | Route path |
| label | string | yes | Navigation label text |
| icon | ReactNode | yes | Icon element |
| isActive | boolean | yes | Whether this is the current page |

#### States
- **Default**: `--ink-secondary` text, `--radius-sm`
- **Hover**: `--surface-inset` background
- **Active**: `--sage-subtle` background, `--sage` text, weight 500
- **Focus**: `--control-focus-ring`

---

### Component: SessionCard (Signature Element)

**Purpose**: Primary booking display surface. Warm, grounded appointment-book entry.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| clientName | string | yes | Client full name |
| clientInitials | string | yes | 1-2 character initials for circle |
| serviceName | string | yes | Name of the booked service |
| date | string | yes | Formatted date string |
| time | string | yes | Formatted time string (monospace) |
| duration | number | yes | Duration in minutes |
| status | "confirmed" \| "cancelled" \| "completed" \| "pending" | yes | Booking status |
| onClick | () => void | no | Click handler for expand/navigate |
| actions | ReactNode | no | Slot for action buttons (Reschedule, Cancel) |

#### States
- **Confirmed**: Left border `--sage`, StatusBadge "Confirmed"
- **Pending**: Left border `--warning`, StatusBadge "Pending"
- **Cancelled**: Left border `--destructive`, StatusBadge "Cancelled"
- **Completed/Past**: Left border `--border`, StatusBadge "Completed"
- **Hover**: `--border-strong` border (when clickable)
- **Focus**: `--control-focus-ring`

#### Surface
- `--surface-1` background, `--border-subtle` border, `--shadow`, `--space-4` padding, `--radius`

#### Children
- `ClientInitials` -- Warm circle with initials
- `StatusBadge` -- Status indicator

#### Accessibility
- Rendered as `<li>` within a list context
- Content order for screen readers: date, time, client name, service, status
- If clickable: `role="button"` or wrapped in focusable element with `aria-label`

---

### Component: SessionCardSkeleton

**Purpose**: Loading placeholder matching SessionCard dimensions.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| count | number | no | Number of skeleton cards to render. Default 3. |

#### States
- **Loading only**: Animated pulse, `--surface-inset` background

---

### Component: StatusBadge

**Purpose**: Visual status indicator with text and color.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| status | "confirmed" \| "cancelled" \| "completed" \| "pending" | yes | Status value |

#### Visual Mapping
- **Confirmed**: `--sage-subtle` bg, `--sage` text
- **Pending**: `--warning-subtle` bg, `--warning` text
- **Cancelled**: `--destructive-subtle` bg, `--destructive` text
- **Completed**: `--surface-inset` bg, `--ink-tertiary` text

#### Sizing
- `text-sm`, weight 500, padding `--space-1` `--space-2`, radius `--radius-full`

#### Accessibility
- Text always visible (never color-only). Screen readers read the status text directly.

---

### Component: ClientInitials

**Purpose**: Warm circle displaying client initials. Part of the signature style.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| initials | string | yes | 1-2 characters |
| size | "sm" \| "md" | no | 28px or 36px. Default "md". |

#### Visual
- `--sage-subtle` background, `--sage` text, weight 600, `--radius-full`
- Size md: 36px, Size sm: 28px

---

### Component: DatePicker

**Purpose**: Custom month calendar grid for date selection. No native date input.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| selectedDate | Date \| null | yes | Currently selected date |
| onDateSelect | (date: Date) => void | yes | Date selection handler |
| availableDates | Date[] | no | Dates that are selectable (all others disabled) |
| minDate | Date | no | Earliest selectable date |
| maxDate | Date | no | Latest selectable date |
| isLoading | boolean | no | Show skeleton state |

#### States
- **Loading**: Skeleton grid placeholder
- **Populated**: Calendar grid with available/unavailable dates
- **Empty**: All dates disabled (no availability)

#### Accessibility
- `role="grid"` with `aria-label="Select a date"`, `aria-roledescription="calendar"`
- Date cells: `role="gridcell"`, `aria-selected` on chosen date, `aria-disabled` on unavailable
- Arrow keys navigate dates, Enter selects
- Month navigation: Previous/Next buttons with `aria-label="Previous month"` / `aria-label="Next month"`
- Today indicator: subtle ring, `aria-current="date"`

---

### Component: TimeSlotGrid

**Purpose**: Grid of available time slot buttons for a selected date.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| slots | { time: string; available: boolean }[] | yes | Available time slots |
| selectedSlot | string \| null | yes | Currently selected time string |
| onSlotSelect | (time: string) => void | yes | Slot selection handler |
| isLoading | boolean | no | Show skeleton state |
| isLocking | boolean | no | Show "Reserving your time..." state after selection |
| lockError | string \| null | no | Error message if lock failed |

#### States
- **Loading**: Skeleton buttons placeholder
- **Populated**: Grid of time buttons
- **Empty**: "No available times" message
- **Locking**: Selected slot shows brief loading indicator
- **Lock Error**: Warning message displayed, slot deselected

#### Slot Button States
- **Available**: `--surface-1` bg, `--border`
- **Hover**: `--sage-ghost` bg, `--border-strong`
- **Selected**: `--sage` bg, white text
- **Unavailable**: Not rendered (excluded from list)

#### Accessibility
- `role="radiogroup"` with `aria-label="Available times"`
- Each slot: `role="radio"` with `aria-checked`
- Time format: monospace, tabular-nums ("9:00 AM")
- Lock status: `aria-live="polite"` region
- Focus moves to first slot when slots load

---

### Component: FormInput

**Purpose**: Styled text input with label, error, and help text.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| label | string | yes | Input label text |
| name | string | yes | Input name attribute |
| type | "text" \| "email" \| "tel" \| "number" | no | Input type. Default "text". |
| value | string | yes | Controlled value |
| onChange | (value: string) => void | yes | Change handler |
| onBlur | () => void | no | Blur handler for validation |
| placeholder | string | no | Placeholder text |
| error | string \| null | no | Error message to display |
| helpText | string | no | Help text below input |
| required | boolean | no | Whether field is required |
| disabled | boolean | no | Disabled state |
| maxLength | number | no | Max character length |
| prefix | ReactNode | no | Prefix element (e.g., URL prefix) |

#### States
- **Default**: `--control-bg` background, `--control-border`
- **Focus**: `--border-focus` + `--control-focus-ring`
- **Error**: `--destructive` border, error message below
- **Disabled**: Opacity 0.5, no pointer events

#### Accessibility
- `<label for="{id}">` associated with input
- `aria-required="true"` when required
- `aria-describedby` linking to error and/or help text
- `aria-invalid="true"` when error present

---

### Component: FormTextarea

**Purpose**: Styled textarea with label, error, help text, and character counter.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| label | string | yes | Textarea label text |
| name | string | yes | Textarea name attribute |
| value | string | yes | Controlled value |
| onChange | (value: string) => void | yes | Change handler |
| onBlur | () => void | no | Blur handler |
| placeholder | string | no | Placeholder text |
| error | string \| null | no | Error message |
| required | boolean | no | Required state |
| disabled | boolean | no | Disabled state |
| maxLength | number | no | Max characters (shows counter when set) |
| rows | number | no | Visible rows. Default 3. |

#### States
- Same as FormInput, plus character counter when `maxLength` is set.

#### Accessibility
- Same as FormInput
- Character count: `aria-live="polite"` region announces remaining characters

---

### Component: FormCheckbox

**Purpose**: Styled checkbox with label text.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| label | string \| ReactNode | yes | Checkbox label (can be long text for consent) |
| name | string | yes | Checkbox name |
| checked | boolean | yes | Controlled checked state |
| onChange | (checked: boolean) => void | yes | Change handler |
| error | string \| null | no | Error message |
| required | boolean | no | Required state |
| disabled | boolean | no | Disabled state |

#### Accessibility
- `<input type="checkbox">` with full label text (not truncated)
- `aria-required="true"` when required
- `aria-describedby` for error
- `aria-invalid="true"` when error present

---

### Component: SelectControl

**Purpose**: Custom select dropdown using Headless UI Listbox. No native `<select>`.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| label | string | yes | Select label text |
| name | string | yes | Select name |
| value | string \| null | yes | Selected value |
| onChange | (value: string) => void | yes | Change handler |
| options | { value: string; label: string; group?: string }[] | yes | Option list, optionally grouped |
| placeholder | string | no | Placeholder text when no selection |
| error | string \| null | no | Error message |
| required | boolean | no | Required state |
| disabled | boolean | no | Disabled state |

#### States
- **Default**: Closed, showing selected value or placeholder
- **Open**: Dropdown visible with `--shadow-md`, `--border`
- **Error**: `--destructive` border

#### Accessibility
- Headless UI Listbox provides full keyboard navigation (arrow keys, type-ahead)
- `aria-expanded`, `aria-haspopup="listbox"`
- Options: `role="option"` with `aria-selected`

---

### Component: RadioGroup

**Purpose**: Styled radio group using Headless UI RadioGroup.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| legend | string | yes | Group label text |
| name | string | yes | Group name |
| value | string | yes | Selected value |
| onChange | (value: string) => void | yes | Change handler |
| options | { value: string; label: string; description?: string }[] | yes | Radio options |
| error | string \| null | no | Error message |

#### Accessibility
- `<fieldset>` with `<legend>`
- Radio inputs with associated labels
- `aria-describedby` for descriptions and errors

---

### Component: ToggleSwitch

**Purpose**: On/off toggle for binary states (e.g., service active/inactive, day enabled/disabled).

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| checked | boolean | yes | Current state |
| onChange | (checked: boolean) => void | yes | Change handler |
| label | string | yes | Accessible label text |
| disabled | boolean | no | Disabled state |

#### Visual
- Active: `--sage` track color
- Inactive: `--ink-muted` track color

#### Accessibility
- `role="switch"` with `aria-checked` and `aria-label`

---

### Component: Modal

**Purpose**: Dialog overlay using Headless UI Dialog. For forms, confirmations.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| isOpen | boolean | yes | Open state |
| onClose | () => void | yes | Close handler (Escape, backdrop click) |
| title | string | yes | Modal heading text |
| children | ReactNode | yes | Modal content |
| size | "sm" \| "md" \| "lg" | no | Width variant. Default "md". |

#### States
- **Opening**: Animate in, trap focus
- **Open**: First focusable element focused
- **Closing**: Animate out, return focus to trigger

#### Visual
- `--shadow-lg`, `--radius-md`, backdrop overlay

#### Accessibility
- `role="dialog"`, `aria-modal="true"`, `aria-labelledby` to title
- Focus trap within modal
- Escape closes modal
- Focus returns to trigger element on close

---

### Component: AlertDialog

**Purpose**: Destructive action confirmation dialog using Headless UI Dialog. Focus defaults to safe action.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| isOpen | boolean | yes | Open state |
| onClose | () => void | yes | Close handler |
| title | string | yes | Dialog heading |
| description | string | yes | Explanation text |
| warning | string \| null | no | Warning callout text (e.g., existing bookings) |
| confirmLabel | string | yes | Destructive action button label |
| cancelLabel | string | no | Safe action button label. Default "Cancel". |
| onConfirm | () => void | yes | Destructive action handler |
| isSubmitting | boolean | no | Loading state for confirm button |

#### States
- **Default**: Focus on cancel/safe button
- **Submitting**: Confirm button loading, form disabled
- **Error**: Error message within dialog

#### Accessibility
- `role="alertdialog"`, `aria-modal="true"`
- Focus moves to safe (cancel) button on open
- Escape closes without confirming

---

### Component: Toast

**Purpose**: Transient notification messages.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| message | string | yes | Toast text |
| variant | "success" \| "error" \| "info" | no | Visual variant. Default "success". |
| duration | number | no | Auto-dismiss time in ms. Default 4000. |

#### States
- **Visible**: Slide in from top-right
- **Dismissed**: Slide out (auto or manual)

#### Accessibility
- `role="status"`, `aria-live="polite"`
- Auto-dismiss respects `prefers-reduced-motion`

---

### Component: EmptyState

**Purpose**: Centered message with optional action for empty data screens.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| message | string | yes | Empty state message |
| action | ReactNode | no | CTA button or link |

#### Visual
- Centered, `--ink-tertiary` text

---

### Component: ErrorBanner

**Purpose**: Error message banner with retry capability.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| message | string | yes | Error message |
| onRetry | () => void | no | Retry handler (shows retry button when provided) |

#### Visual
- `--destructive-subtle` background, `--destructive` text

#### Accessibility
- `role="alert"` for immediate announcement

---

### Component: LoadingSkeleton

**Purpose**: Generic animated loading placeholder.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| variant | "text" \| "card" \| "stat" \| "input" \| "grid" | yes | Shape variant |
| count | number | no | Number of skeleton items. Default 1. |

#### Visual
- `--surface-inset` background with animated pulse

---

### Component: Button

**Purpose**: Primary interactive button with variants.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| variant | "primary" \| "secondary" \| "ghost" \| "destructive" | no | Visual variant. Default "primary". |
| size | "sm" \| "md" | no | Size variant. Default "md". |
| children | ReactNode | yes | Button content |
| onClick | () => void | no | Click handler |
| type | "button" \| "submit" | no | Button type. Default "button". |
| disabled | boolean | no | Disabled state |
| isLoading | boolean | no | Loading state (shows spinner, disables interaction) |
| fullWidth | boolean | no | Full-width button |
| asChild | boolean | no | Render as child element (for link buttons) |

#### Visual Variants
- **Primary**: `--sage` bg, white text, `--shadow-sm`, hover `--sage-hover`
- **Secondary**: `--surface-1` bg, `--ink` text, `--border`, hover `--surface-2`
- **Ghost**: transparent bg, `--ink-secondary` text, hover `--surface-inset`
- **Destructive**: `--destructive` bg, white text, hover darken

#### States
- **Default**, **Hover**, **Active/Pressed**, **Focus** (`--control-focus-ring`), **Disabled** (opacity 0.5), **Loading** (spinner + disabled)

#### Accessibility
- `aria-disabled` when loading or disabled (not removed from DOM)
- Loading state: `aria-busy="true"`

---

### Component: StepProgress

**Purpose**: Horizontal step indicator for multi-step flows (setup wizard, create booking modal).

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| steps | { label: string; status: "completed" \| "current" \| "upcoming" }[] | yes | Step definitions |
| currentStep | number | yes | Current step index (0-based) |

#### Visual
- Completed: Checkmark icon, `--sage` color
- Current: Filled circle, `--sage` color
- Upcoming: Empty circle, `--ink-muted` color

#### Accessibility
- `aria-label="Setup progress: Step {n} of {total}, {step name}"`
- `role="progressbar"` with `aria-valuenow` and `aria-valuemax`

---

### Component: CopyButton

**Purpose**: Copy-to-clipboard button with confirmation feedback.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| text | string | yes | Text to copy |
| label | string | no | Button label. Default "Copy". |

#### States
- **Default**: Copy icon
- **Copied**: Checkmark icon + "Copied" text (reverts after 2s)

#### Accessibility
- `aria-label="Copy {context}"` (e.g., "Copy booking page URL")
- `aria-live="polite"` announces "Copied" on success

---

### Component: FilterTabs

**Purpose**: Tab-style filter for list views (Upcoming / Past / Cancelled).

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| tabs | { value: string; label: string }[] | yes | Tab definitions |
| activeTab | string | yes | Current active tab value |
| onChange | (value: string) => void | yes | Tab change handler |

#### Visual
- Active: `--sage` underline, `--ink` text
- Inactive: `--ink-tertiary` text, hover `--ink-secondary`

#### Accessibility
- `role="tablist"` with `role="tab"` per tab
- `aria-selected` on active tab
- Arrow keys navigate between tabs

---

### Component: BookingSummaryCard

**Purpose**: Read-only booking details display used in forms and confirmation screens.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| serviceName | string | yes | Service name |
| date | string | yes | Formatted date |
| time | string | yes | Formatted time |
| duration | number | yes | Duration in minutes |
| modality | "online" \| "in_person" | no | Session modality |
| clientName | string | no | Client name (when shown in therapist context) |

#### Visual
- `--surface-inset` background, read-only presentation

---

### Component: Popover

**Purpose**: Small floating panel anchored to a trigger element. Used for time block editing.

#### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| isOpen | boolean | yes | Open state |
| onClose | () => void | yes | Close handler |
| trigger | ReactNode | yes | Anchor element |
| children | ReactNode | yes | Popover content |

#### States
- **Open**: `--shadow-md`, focus trapped
- **Closed**: Hidden

#### Accessibility
- `role="dialog"`, `aria-label` for context
- Focus trap within popover
- Escape closes, focus returns to trigger
- Closes on click outside

---

## 2. Feature-Level Components

---

### Feature: Therapist Dashboard (ux-flows.md Section 1)

---

#### Component: DashboardPage

**Source flow**: Section 1 -- Dashboard Home
**Route**: `app/(auth)/dashboard/page.tsx` (Server Component)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| -- | -- | -- | Page-level component, no external props |

##### States
- **Loading**: PageLayout + SessionCardSkeleton(count=3) + LoadingSkeleton(variant="stat", count=3)
- **Empty**: PageLayout + EmptyState("No upcoming sessions") + CTA buttons ("Set up your services" or "Share your booking page")
- **Populated**: PageLayout + QuickStats + upcoming SessionCard list + booking page link
- **Error**: PageLayout + ErrorBanner with retry

##### Children
- `PageLayout` -- Shell with title "Dashboard"
- `QuickStats` -- Today's sessions, this week total, next available slot
- `SessionCard` -- Repeated for upcoming bookings (next 7 days)
- `CopyButton` -- Copy booking page URL
- `Toast` -- Confirms URL copied

##### Data Requirements
- `useUpcomingBookings()` -- Fetches bookings for next 7 days (server state)
- `useQuickStats()` -- Fetches today count, week total, next available slot (server state)
- Therapist slug for booking URL (from auth session)

##### State Management
- Server state via data fetching hooks (React Query or SWR)
- No local state beyond copy confirmation

---

#### Component: QuickStats

**Source flow**: Section 1 -- Quick stats row

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| todayCount | number | yes | Sessions today |
| weekTotal | number | yes | Sessions this week |
| nextSlot | string \| null | yes | Next available slot formatted string, or null |
| isLoading | boolean | no | Loading state |

##### Accessibility
- Each stat: `aria-label` on value (e.g., `aria-label="3 sessions today"`)
- Labels: `--ink-tertiary`, `text-sm`, uppercase
- Values: `--ink`, `text-xl`, tabular-nums

---

### Feature: Therapist Setup (ux-flows.md Section 2)

---

#### Component: SetupWizardPage

**Source flow**: Section 2 -- Therapist Setup Flow
**Route**: `app/setup/page.tsx` (Client Component -- wizard state management)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| -- | -- | -- | Page-level component, no external props |

##### States
- **Loading**: Full-page centered spinner while determining current step
- **Step 1-5**: Renders appropriate step component based on `currentStep`
- **Error**: ErrorBanner for step load failure

##### Children
- `StepProgress` -- Horizontal step indicator (5 steps)
- `SetupPracticeDetails` -- Step 1
- `SetupGoogleCalendar` -- Step 2
- `SetupFirstService` -- Step 3
- `SetupAvailability` -- Step 4
- `SetupComplete` -- Step 5

##### Data Requirements
- `useTherapistSetup()` -- Reads current `setup_step` to determine resume point
- Server actions for each step mutation

##### State Management
- Local state: `currentStep` (number), synced with server `setup_step`
- Form state: Per-step local form state
- URL state: Not needed (step tracked server-side for resume)

---

#### Component: SetupPracticeDetails

**Source flow**: Section 2 -- Step 1: Practice Details

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| defaultName | string | yes | Pre-filled name from Google profile |
| onComplete | () => void | yes | Advances wizard to next step |

##### States
- **Default**: Form with pre-filled name
- **Validating**: Inline errors on blur
- **Checking Slug**: Debounced uniqueness check (loading indicator on slug field)
- **Slug Available**: Green checkmark indicator
- **Slug Taken**: Inline error on slug field
- **Submitting**: "Continue" button loading, form disabled
- **Error**: Inline field errors + ErrorBanner for server errors

##### Children
- `FormInput` -- Practice name (required, max 100 chars)
- `SelectControl` -- Timezone (IANA list, grouped by region, auto-detected default)
- `FormInput` -- Slug (with URL preview prefix, debounced uniqueness check)
- `Button` -- "Continue" (primary)

##### Data Requirements
- Server Action: `savePracticeDetails({ practiceName, timezone, slug })`
- Server Action: `checkSlugAvailability(slug)` -- debounced 500ms

##### State Management
- Local form state (controlled inputs)
- Slug uniqueness: server state with debounced query

##### Accessibility
- Slug uniqueness result: `aria-live="polite"` region
- URL preview: `aria-label` describes full booking URL

---

#### Component: SetupGoogleCalendar

**Source flow**: Section 2 -- Step 2: Connect Google Calendar

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| onComplete | () => void | yes | Advances wizard to next step |

##### States
- **Default**: Explanation text + "Connect Google Calendar" button
- **Connecting**: Button disabled, loading indicator, "Connecting..." text
- **Success**: Checkmark + "Google Calendar connected", auto-advance or Continue button
- **Error**: Error message + "Try again" button
- **Denied**: Specific message about required Calendar access + "Try again"

##### Children
- `Button` -- "Connect Google Calendar" (primary, with Google icon)
- `Button` -- "Try again" (shown on error/denied)

##### Data Requirements
- Initiates OAuth redirect for Calendar scopes (incremental authorization)
- Reads OAuth callback result from URL params on return

##### State Management
- Local state: connection status
- Server state: OAuth token storage (handled server-side)

##### Accessibility
- `aria-live="assertive"` for connection status announcements
- Focus returns to "Try again" after OAuth redirect failure

---

#### Component: SetupFirstService

**Source flow**: Section 2 -- Step 3: Create First Service

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| onComplete | () => void | yes | Advances wizard to next step |

##### States
- **Default**: Service creation form (empty fields)
- **Validating**: Inline errors on blur
- **Submitting**: "Create & Continue" button loading
- **Success**: Brief confirmation, auto-advance
- **Error**: Inline field errors + ErrorBanner

##### Children
- `FormInput` -- Service name (required, max 100 chars)
- `SelectControl` -- Duration (15, 30, 45, 60, 75, 90, 120 min)
- `RadioGroup` -- Modality: Online / In-Person (with contextual notes)
- `FormTextarea` -- Description (optional, max 500 chars, character counter)
- `Button` -- "Create & Continue" (primary)

##### Data Requirements
- Server Action: `createService({ name, duration, modality, description })`

##### State Management
- Local form state

---

#### Component: SetupAvailability

**Source flow**: Section 2 -- Step 4: Set Availability

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| timezone | string | yes | Therapist timezone (set in Step 1) |
| onComplete | () => void | yes | Advances wizard to next step |

##### States
- **Default**: Weekly grid with no blocks set
- **Editing**: Active day expanded with time block inputs
- **Validating**: Overlap errors shown inline
- **Submitting**: "Complete Setup" button loading
- **Error**: Overlap error per-block or ErrorBanner

##### Children
- `DayToggleRow` -- 7-day row (Sun-Sat), toggle each on/off
- `TimeBlockEditor` -- Per active day: list of start/end time selects
- `FormInput` -- Minimum notice hours (number, default 24)
- `Button` -- "Complete Setup" (primary)

##### Data Requirements
- Server Action: `saveAvailability({ blocks: [{ dayOfWeek, startTime, endTime }], minimumNoticeHours })`
- Server Action: `completeSetup()` -- marks setup complete

##### State Management
- Local state: day toggles, time block arrays, minimum notice
- Validation: client-side overlap detection

---

#### Component: SetupComplete

**Source flow**: Section 2 -- Step 5: Setup Complete

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| bookingUrl | string | yes | Full booking page URL |
| slug | string | yes | Therapist slug |

##### States
- **Populated only**: Always shown after successful setup

##### Children
- `CopyButton` -- Copy booking URL
- `Button` -- "Preview Booking Page" (secondary, opens new tab)
- `Button` -- "Go to Dashboard" (primary)

##### Accessibility
- Focus auto-moves to success heading on arrival (`tabindex="-1"`, auto-focus)

---

### Feature: Services Management (ux-flows.md Section 3)

---

#### Component: ServicesPage

**Source flow**: Section 3 -- Service List
**Route**: `app/(auth)/services/page.tsx` (Server Component)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| -- | -- | -- | Page-level component, no external props |

##### States
- **Loading**: PageLayout + LoadingSkeleton(variant="card", count=3)
- **Empty**: PageLayout + EmptyState("No services yet") + "Create your first service" CTA
- **Populated**: PageLayout + ServiceCard list
- **Error**: PageLayout + ErrorBanner with retry

##### Children
- `PageLayout` -- Shell with title "Services", action "Add Service" button
- `ServiceCard` -- Repeated for each service
- `ServiceFormModal` -- Create/Edit service modal
- `AlertDialog` -- Delete confirmation dialog
- `Toast` -- Action confirmations

##### Data Requirements
- `useServices()` -- Fetches therapist's services (server state)
- Server Action: `toggleServiceActive(serviceId, isActive)` -- Toggle active/inactive
- Server Action: `deleteService(serviceId)` -- Delete service

##### State Management
- Server state: service list (refetched after mutations)
- Local state: modal open/close, selected service for edit/delete

---

#### Component: ServiceCard

**Source flow**: Section 3 -- Service card within list

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| id | string | yes | Service ID |
| name | string | yes | Service name |
| duration | number | yes | Duration in minutes |
| modality | "online" \| "in_person" | yes | Modality |
| description | string \| null | no | Description (truncated to 1 line) |
| isActive | boolean | yes | Active status |
| onToggleActive | (isActive: boolean) => void | yes | Toggle handler |
| onEdit | () => void | yes | Edit handler |
| onDelete | () => void | yes | Delete handler |

##### States
- **Active**: Normal display
- **Inactive**: Muted appearance (lower opacity on name/description)

##### Children
- `ToggleSwitch` -- Active/Inactive toggle
- `Button` (ghost) -- Edit action
- `Button` (ghost, destructive hover) -- Delete action

##### Accessibility
- `<li>` within `<ul aria-label="Your services">`
- Toggle: `role="switch"`, `aria-checked`, `aria-label="Active status for {name}"`
- Edit/Delete: `aria-label="Edit {name}"` / `aria-label="Delete {name}"`

---

#### Component: ServiceFormModal

**Source flow**: Section 3 -- Create/Edit Service

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| isOpen | boolean | yes | Open state |
| onClose | () => void | yes | Close handler |
| service | { id: string; name: string; duration: number; modality: string; description?: string } \| null | no | Existing service for edit mode; null for create mode |
| onSave | () => void | yes | Success callback (refreshes list) |

##### States
- **Default**: Empty form (create) or pre-filled (edit)
- **Validating**: Inline errors on blur
- **Submitting**: Save button loading, form disabled
- **Success**: Modal closes, triggers onSave
- **Error**: Inline field errors + ErrorBanner in modal

##### Children
- `Modal` -- Dialog shell
- `FormInput` -- Service name (required, max 100 chars)
- `SelectControl` -- Duration (15, 30, 45, 60, 75, 90, 120 min)
- `RadioGroup` -- Modality: Online / In-Person
- `FormTextarea` -- Description (optional, max 500 chars)
- `Button` -- "Save" (primary) / "Cancel" (secondary)

##### Data Requirements
- Server Action: `createService({ name, duration, modality, description })`
- Server Action: `updateService(id, { name, duration, modality, description })`

##### State Management
- Local form state (controlled inputs)

---

#### Component: DeleteServiceDialog

**Source flow**: Section 3 -- Delete Confirmation

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| isOpen | boolean | yes | Open state |
| onClose | () => void | yes | Close handler |
| serviceName | string | yes | Service name for display |
| bookingCount | number | yes | Number of existing bookings (0 if none) |
| onConfirm | () => void | yes | Delete handler |
| isSubmitting | boolean | no | Loading state |

##### States
- **Default**: Confirmation prompt
- **Has bookings**: Additional warning callout about existing bookings
- **Submitting**: Delete button loading
- **Error**: Error message within dialog

##### Children
- `AlertDialog` -- Shell with destructive confirm

---

### Feature: Availability Management (ux-flows.md Section 4)

---

#### Component: AvailabilityPage

**Source flow**: Section 4 -- Weekly Availability View
**Route**: `app/(auth)/availability/page.tsx` (Server Component wrapper, client island for interactive grid)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| -- | -- | -- | Page-level component, no external props |

##### States
- **Loading**: PageLayout + 7-day skeleton grid
- **Empty**: PageLayout + all days inactive + "Set your first available hours" prompt
- **Populated**: PageLayout + weekly grid with time blocks
- **Error**: PageLayout + ErrorBanner with retry

##### Children
- `PageLayout` -- Shell with title "Availability" (fullWidth=true)
- `TimezoneDisplay` -- Read-only timezone badge
- `MinimumNoticeInput` -- Inline-save number input
- `WeeklyAvailabilityGrid` -- Interactive 7-day grid
- `TimeBlockPopover` -- Edit/add time block popover
- `Toast` -- Save confirmations and errors

##### Data Requirements
- `useAvailability()` -- Fetches therapist's availability blocks (server state)
- `useTherapistProfile()` -- Timezone and minimum notice hours
- Server Action: `saveAvailabilityBlock({ dayOfWeek, startTime, endTime })`
- Server Action: `deleteAvailabilityBlock(blockId)`
- Server Action: `updateMinimumNotice(hours)`

##### State Management
- Server state: availability blocks (refetched after mutations)
- Local state: active popovers, dirty state tracking, optimistic updates

---

#### Component: WeeklyAvailabilityGrid

**Source flow**: Section 4 -- Weekly grid layout

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| blocks | { id: string; dayOfWeek: number; startTime: string; endTime: string }[] | yes | Availability blocks |
| onDayToggle | (dayOfWeek: number, enabled: boolean) => void | yes | Day toggle handler |
| onBlockClick | (block: { id: string; dayOfWeek: number; startTime: string; endTime: string }) => void | yes | Block click handler (opens edit popover) |
| onAddBlock | (dayOfWeek: number) => void | yes | Add block handler (opens add popover) |
| isLoading | boolean | no | Loading state |

##### States
- **Loading**: 7-column skeleton
- **Populated**: Days with time blocks as vertical bars

##### Day Column States
- **Active**: Time blocks displayed, `--sage-subtle` styling on blocks
- **Inactive**: `--surface-inset` bg, `--ink-muted` "Not available" text

##### Accessibility
- `role="grid"` with `aria-label="Weekly availability schedule"`
- Day toggles: `role="switch"` with `aria-checked`, `aria-label="Available on {day}"`
- Blocks: Focusable, `aria-label="{day} {start} to {end}"`, Enter to edit
- Arrow keys navigate between days, Tab into blocks within a day

##### Responsive
- Desktop: 7-column grid
- Tablet: 7-column compressed or horizontal scroll
- Mobile: Stacked list view, one day per row, expandable

---

#### Component: TimeBlockPopover

**Source flow**: Section 4 -- Edit Time Block (Popover)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| isOpen | boolean | yes | Open state |
| onClose | () => void | yes | Close handler |
| block | { id?: string; dayOfWeek: number; startTime?: string; endTime?: string } \| null | yes | Existing block for edit, or day info for new block |
| onSave | (block: { dayOfWeek: number; startTime: string; endTime: string }) => void | yes | Save handler |
| onDelete | (blockId: string) => void | no | Delete handler (edit mode only) |
| existingBlocks | { startTime: string; endTime: string }[] | yes | Other blocks on same day (for overlap validation) |

##### States
- **Default**: Start/end time selects (pre-filled for edit, empty for add)
- **Validating**: Overlap warning shown
- **Saving**: Brief loading state
- **Error**: Inline error for overlap or invalid range

##### Children
- `SelectControl` -- Start time (15-min increments, 6:00 AM - 11:45 PM)
- `SelectControl` -- End time (must be after start)
- `Button` (ghost, destructive) -- "Remove" (edit mode only)

##### Accessibility
- Popover: `role="dialog"`, `aria-label="Edit time block"`
- Focus trap within popover
- Escape closes, focus returns to trigger

---

#### Component: TimezoneDisplay

**Purpose**: Read-only badge showing current timezone.

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| timezone | string | yes | IANA timezone identifier |
| label | string | yes | Display label (e.g., "America/Toronto (ET)") |

---

#### Component: MinimumNoticeInput

**Purpose**: Inline-save number input for minimum booking notice hours.

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| value | number | yes | Current hours value |
| onSave | (hours: number) => void | yes | Save handler (triggered on blur or explicit save) |

##### Children
- `FormInput` (type="number") with "hours minimum notice" suffix label

---

### Feature: Public Booking Page (ux-flows.md Section 5)

---

#### Component: BookingPage

**Source flow**: Section 5 -- Public Booking Page
**Route**: `app/[slug]/booking/page.tsx` (Server Component for initial service list)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| -- | -- | -- | Page-level route component. Slug from URL params. |

##### Layout
- No sidebar. Single-column, max-width 640px, centered. Mobile-first.

##### States
- **Loading**: Skeleton service cards
- **Empty (no active services)**: "Online booking is not yet available. Please contact {practice name} directly."
- **Populated**: Multi-step booking flow
- **Error**: Full-page error with retry link
- **Invalid slug**: 404 page

##### Children
- `BookingLayout` -- Clean single-column shell
- `ServiceSelection` -- Step 1: Select service
- `DateTimeSelection` -- Step 2: Select date and time
- `BookingDetailsForm` -- Step 3: Client details form
- `BookingConfirmation` -- Step 4: Confirmation

##### Data Requirements
- Server Component: Fetches therapist profile + active services via slug (public RLS read)
- Client-side: Slot fetching, slot locking, booking creation

##### State Management
- URL state: Current step could use URL search params for back-button support
- Local state: selected service, selected date, selected slot, form values
- Server state: slots (fetched per date), slot lock, booking creation

---

#### Component: BookingLayout

**Purpose**: Clean single-column layout for public and client-management pages.

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| practiceName | string | yes | Practice name for header |
| subtitle | string | no | Subtitle text (e.g., "Book a Session") |
| children | ReactNode | yes | Page content |

##### Visual
- Max-width 640px, centered, no sidebar, `--canvas` background

---

#### Component: ServiceSelection

**Source flow**: Section 5 -- Service Selection

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| services | { id: string; name: string; duration: number; modality: string; description?: string }[] | yes | Active services |
| onSelect | (serviceId: string) => void | yes | Service selection handler |
| timezone | string | yes | Therapist timezone for display |

##### States
- **Loading**: Skeleton cards (2-3)
- **Empty**: "Online booking is not yet available" message
- **Populated**: Service cards for selection

##### Children
- `PublicServiceCard` -- Clickable card per service

##### Accessibility
- Cards: `role="radio"` within `role="radiogroup"` with `aria-label="Select a service"`
- Keyboard: Tab between services, Enter/Space to select
- Focus indicator: `--control-focus-ring` on card

---

#### Component: PublicServiceCard

**Purpose**: Client-facing service display card (clickable for selection).

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| name | string | yes | Service name |
| duration | number | yes | Duration in minutes |
| modality | "online" \| "in_person" | yes | Session modality |
| description | string \| null | no | Optional description |
| isSelected | boolean | no | Selection state |
| onClick | () => void | yes | Click handler |

##### States
- **Default**: `--surface-1` bg, `--border-subtle`
- **Hover**: `--border-strong` border
- **Selected**: `--sage` border or accent indicator
- **Focus**: `--control-focus-ring`

---

#### Component: DateTimeSelection

**Source flow**: Section 5 -- Date & Time Selection

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| serviceId | string | yes | Selected service ID |
| serviceName | string | yes | Selected service name |
| duration | number | yes | Service duration |
| therapistSlug | string | yes | For API calls |
| timezone | string | yes | Therapist timezone |
| onSlotSelected | (date: string, time: string) => void | yes | Slot lock success handler |
| onBack | () => void | yes | Back to service selection |

##### States
- **Loading**: Calendar skeleton + "Loading available times..." text
- **Empty (no slots)**: "No available times for this service" message
- **Populated**: Calendar + time slots
- **Error**: "Unable to load available times" with retry
- **Slot taken (race condition)**: Warning message, slot deselected

##### Children
- `Button` (ghost) -- "Back to services"
- `BookingSummaryCard` -- Selected service summary
- `DatePicker` -- Month calendar
- `TimeSlotGrid` -- Available times for selected date

##### Data Requirements
- `useAvailableSlots(therapistSlug, serviceId, date)` -- Fetches slots for selected date
- Server Action: `lockSlot({ therapistSlug, serviceId, startTime, endTime })` -- Creates slot lock

##### State Management
- Local state: selected date
- Server state: available slots (refetched on date change), slot lock result

---

#### Component: BookingDetailsForm

**Source flow**: Section 5 -- Booking Details Form

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| serviceName | string | yes | For summary display |
| date | string | yes | Selected date |
| time | string | yes | Selected time |
| duration | number | yes | Duration |
| practiceName | string | yes | For consent text |
| slotLockId | string | yes | Slot lock reference |
| lockExpiresAt | string | yes | Lock expiration time |
| therapistSlug | string | yes | For booking creation |
| serviceId | string | yes | Service ID |
| onSuccess | (bookingDetails: BookingResult) => void | yes | Success handler |
| onBack | () => void | yes | Back to time selection (releases lock) |
| onLockExpired | () => void | yes | Lock expired handler |

##### States
- **Default**: Empty form with booking summary at top
- **Validating**: Inline errors on blur
- **Submitting**: "Confirm Booking" button loading, form disabled
- **Slot expired**: Warning message + back button
- **Error (server)**: ErrorBanner + form stays populated
- **Error (slot taken)**: "Time no longer available" + back link

##### Children
- `BookingSummaryCard` -- Service, date, time, duration (read-only)
- `FormInput` -- Full name (required)
- `FormInput` -- Email (required, format validation)
- `FormInput` -- Phone (required, Canadian 10-digit or international E.164)
- `FormCheckbox` -- Consent (required, unchecked default, full text)
- `Button` -- "Confirm Booking" (primary)
- `Button` (ghost) -- "Back"
- Slot hold timer display (`--ink-tertiary`)

##### Data Requirements
- Server Action: `createBooking({ slotLockId, serviceId, therapistSlug, clientName, clientEmail, clientPhone, clientConsent, startTime, endTime })`
- Server Action: `releaseSlotLock(slotLockId)` -- On back navigation

##### State Management
- Local form state
- Timer: Countdown to lock expiration (local interval)

##### Accessibility
- Consent: full label text, `aria-required="true"`
- Phone: `aria-describedby` to format hint
- Submit disabled: `aria-disabled` (not removed from DOM)
- Focus on first error field after failed submission

---

#### Component: BookingConfirmation

**Source flow**: Section 5 -- Booking Confirmation

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| serviceName | string | yes | Service name |
| date | string | yes | Formatted date |
| time | string | yes | Formatted time |
| duration | number | yes | Duration |
| modality | "online" \| "in_person" | yes | Session modality |
| clientEmail | string | yes | Client email (for confirmation message) |
| timezone | string | yes | Timezone label |

##### States
- **Populated only**: Terminal state after successful booking

##### Children
- Success icon (checkmark, `--sage`)
- `BookingSummaryCard` -- Full booking details
- Modality note: "A Google Meet link will be sent to your email" (online) or "In-Person" (in-person)
- Confirmation email notice

##### Accessibility
- Focus moves to "Booking Confirmed" heading on arrival (`tabindex="-1"`)
- `role="status"` on confirmation container

---

### Feature: Booking Management -- Therapist (ux-flows.md Section 6)

---

#### Component: BookingsPage

**Source flow**: Section 6 -- Booking List
**Route**: `app/(auth)/bookings/page.tsx` (Server Component)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| -- | -- | -- | Page-level component, no external props |

##### States
- **Loading**: PageLayout + SessionCardSkeleton(count=5)
- **Empty**: PageLayout + EmptyState("No bookings yet. Share your booking page to start receiving appointments.") + CopyButton for link
- **Populated**: PageLayout + filtered SessionCard list grouped by date
- **Error**: PageLayout + ErrorBanner with retry

##### Children
- `PageLayout` -- Shell with title "Bookings", action "Create Booking" button
- `FilterTabs` -- Upcoming / Past / Cancelled
- `DateGroupHeader` -- Date labels between card groups
- `SessionCard` -- Repeated per booking (with actions slot)
- `CreateBookingModal` -- Multi-step booking creation
- `RescheduleModal` -- Reschedule flow
- `CancelBookingDialog` -- Cancellation confirmation
- `Toast` -- Action confirmations

##### Data Requirements
- `useBookings(filter: "upcoming" | "past" | "cancelled")` -- Fetches filtered bookings (server state)

##### State Management
- URL state: Active filter tab (search param for shareable URLs)
- Server state: booking list (refetched after mutations)
- Local state: modals open/close, selected booking

---

#### Component: DateGroupHeader

**Purpose**: Date label between groups of bookings.

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| date | string | yes | Formatted date ("Today", "Tomorrow", "Wed, Feb 12") |

##### Visual
- `--ink-tertiary`, text-sm, uppercase

---

#### Component: CreateBookingModal

**Source flow**: Section 6 -- Create Booking (Modal)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| isOpen | boolean | yes | Open state |
| onClose | () => void | yes | Close handler |
| onSuccess | () => void | yes | Success callback (refreshes list) |

##### States
- **Step 1**: Select service (radio list of active services)
- **Step 2**: Select date & time (DatePicker + TimeSlotGrid)
- **Step 3**: Client details (name, email, phone, consent)
- **Submitting**: "Create Booking" button loading
- **Success**: Modal closes, triggers onSuccess
- **Error**: Inline errors + ErrorBanner

##### Children
- `Modal` -- Dialog shell (size="lg")
- `StepProgress` -- Step indicator (3 steps)
- `RadioGroup` -- Service selection (Step 1)
- `DatePicker` + `TimeSlotGrid` -- Slot selection (Step 2)
- `FormInput` (x3) + `FormCheckbox` -- Client details (Step 3)
- `Button` -- Step-appropriate primary action / "Cancel"

##### Data Requirements
- `useServices()` -- Active services list
- `useAvailableSlots(serviceId, date)` -- Slots for selected date
- Server Action: `lockSlot(...)` -- Lock selected slot
- Server Action: `createBooking(...)` -- Create booking (therapist as actor)

##### State Management
- Local state: current step, selected service, selected date/slot, form values

---

#### Component: RescheduleModal

**Source flow**: Section 6 -- Reschedule Flow (Modal)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| isOpen | boolean | yes | Open state |
| onClose | () => void | yes | Close handler |
| booking | { id: string; serviceName: string; serviceId: string; clientName: string; date: string; time: string; duration: number } | yes | Current booking details |
| onSuccess | () => void | yes | Success callback |

##### States
- **Default**: Current booking summary + date/time picker
- **Slots loading**: Skeleton
- **Submitting**: "Confirm Reschedule" button loading
- **Success**: Modal closes, triggers onSuccess
- **Error (slot unavailable)**: Inline error
- **Error (server)**: ErrorBanner

##### Children
- `Modal` -- Dialog shell
- `BookingSummaryCard` -- Current booking info (read-only)
- `DatePicker` + `TimeSlotGrid` -- New slot selection
- `Button` -- "Confirm Reschedule" (primary) / "Cancel" (secondary)

##### Data Requirements
- `useAvailableSlots(serviceId, date)` -- Slots for selected date
- Server Action: `lockSlot(...)` -- Lock new slot
- Server Action: `rescheduleBooking(bookingId, { newStartTime, newEndTime, slotLockId })`

##### State Management
- Local state: selected date, selected slot

---

#### Component: CancelBookingDialog

**Source flow**: Section 6 -- Cancel Booking (Dialog)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| isOpen | boolean | yes | Open state |
| onClose | () => void | yes | Close handler |
| booking | { id: string; clientName: string; serviceName: string; date: string; time: string } | yes | Booking to cancel |
| onSuccess | () => void | yes | Success callback |

##### States
- **Default**: Confirmation prompt with booking summary
- **Submitting**: "Cancel Booking" button loading
- **Success**: Dialog closes, triggers onSuccess
- **Error**: Error message within dialog

##### Children
- `AlertDialog` -- Shell with "Cancel Booking" as destructive action, "Keep Booking" as safe action
- `FormTextarea` -- Cancellation reason (optional)

##### Data Requirements
- Server Action: `cancelBooking(bookingId, { cancellationReason? })`

---

### Feature: Client Booking Management (ux-flows.md Section 7)

---

#### Component: ManageBookingPage

**Source flow**: Section 7 -- Client Booking Management
**Route**: `app/manage/[token]/page.tsx` (Server Component)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| -- | -- | -- | Page-level route component. Token from URL params. |

##### Layout
- Same as public booking page: `BookingLayout`, single-column, 640px max-width, centered.

##### States
- **Loading**: Skeleton card
- **Populated (confirmed)**: Booking details + Reschedule/Cancel actions
- **Invalid/expired token**: "This link is no longer valid" message
- **Already cancelled**: Booking details (read-only) + "Cancelled" badge, no actions
- **Already completed**: Booking details (read-only) + "Completed" badge, no actions
- **Error**: "Unable to load your booking" with retry

##### Children
- `BookingLayout` -- Clean single-column shell
- `ClientBookingDetail` -- Booking information display
- `ClientRescheduleFlow` -- Reschedule view
- `ClientCancelFlow` -- Cancellation confirmation

##### Data Requirements
- Server Component: Fetches booking by manage_token (public RLS read scoped to token)

##### State Management
- Local state: current view (detail / reschedule / cancel)
- Server state: booking data, slot fetching

---

#### Component: ClientBookingDetail

**Source flow**: Section 7 -- Booking Detail (Client View)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| booking | { serviceName: string; date: string; time: string; duration: number; modality: string; status: string; meetLink?: string } | yes | Booking data |
| onReschedule | () => void | no | Reschedule handler (only for confirmed bookings) |
| onCancel | () => void | no | Cancel handler (only for confirmed bookings) |

##### Children
- `BookingSummaryCard` -- Full booking details
- `StatusBadge` -- Current status
- `Button` (secondary) -- "Reschedule" (confirmed only)
- `Button` (ghost, destructive) -- "Cancel Booking" (confirmed only)

##### Accessibility
- Buttons: `aria-label` with context (e.g., "Reschedule your {service name} appointment")

---

#### Component: ClientRescheduleFlow

**Source flow**: Section 7 -- Client Reschedule Flow

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| booking | { id: string; serviceName: string; serviceId: string; date: string; time: string; duration: number } | yes | Current booking |
| manageToken | string | yes | Client manage token |
| therapistSlug | string | yes | For slot fetching |
| timezone | string | yes | Therapist timezone |
| onSuccess | (updatedBooking: object) => void | yes | Success handler |
| onBack | () => void | yes | Return to detail view |

##### States
- **Default**: Date/time picker with current booking summary
- **Loading slots**: Skeleton
- **Submitting**: "Confirm New Time" button loading
- **Success**: Confirmation with updated details
- **Slot unavailable**: Warning + pick different time
- **Error**: ErrorBanner

##### Children
- `BookingSummaryCard` -- Current booking (read-only)
- `DatePicker` + `TimeSlotGrid` -- New slot selection
- `Button` -- "Confirm New Time" (primary) / "Go Back" (ghost)

##### Data Requirements
- `useAvailableSlots(therapistSlug, serviceId, date)` -- Public slot fetch
- Server Action: `clientRescheduleBooking(manageToken, { newStartTime, newEndTime })`

---

#### Component: ClientCancelFlow

**Source flow**: Section 7 -- Client Cancel Flow (Inline Confirmation)

##### Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| booking | { id: string; serviceName: string; date: string; time: string } | yes | Booking to cancel |
| manageToken | string | yes | Client manage token |
| onSuccess | () => void | yes | Success handler |
| onBack | () => void | yes | Return to detail view |

##### States
- **Default**: Inline confirmation prompt (not a modal)
- **Submitting**: "Confirm Cancellation" button loading
- **Success**: Updated view showing "Cancelled" status
- **Error**: Error message with retry

##### Children
- Confirmation text: "Are you sure you want to cancel your {service name} appointment on {date} at {time}?"
- `Button` -- "Confirm Cancellation" (destructive)
- `Button` -- "Go Back" (secondary)

##### Data Requirements
- Server Action: `clientCancelBooking(manageToken)`

##### Accessibility
- `aria-live="assertive"` for cancellation result
- Focus moves to result heading after action

---

## 3. Route Structure

```
app/
  layout.tsx                          # Root layout (fonts, global styles)
  (auth)/                             # Therapist authenticated route group
    layout.tsx                        # PageLayout shell (sidebar + main)
    dashboard/
      page.tsx                        # DashboardPage (Server Component)
    services/
      page.tsx                        # ServicesPage (Server Component)
    availability/
      page.tsx                        # AvailabilityPage (Server Component)
    bookings/
      page.tsx                        # BookingsPage (Server Component)
  [slug]/
    booking/
      page.tsx                        # BookingPage (Server Component for SSR services)
  manage/
    [token]/
      page.tsx                        # ManageBookingPage (Server Component)
  setup/
    page.tsx                          # SetupWizardPage (Client Component for wizard state)
  api/
    auth/
      callback/
        route.ts                      # Google OAuth callback handler
    slots/
      route.ts                        # Public slot availability endpoint
    booking/
      route.ts                        # Public booking creation endpoint
```

**Route count**: 8 page routes + 3 API routes = 11 routes total

---

## 4. Data Hooks & Server Actions

### Data Fetching Hooks (Client-Side, Server State)

All hooks use server state management (React Query or SWR pattern) with automatic cache invalidation on mutations.

| Hook | Purpose | Auth | Caching |
|------|---------|------|---------|
| `useTherapistProfile()` | Therapist profile, timezone, slug | Authenticated (Supabase client) | Stale-while-revalidate |
| `useTherapistSetup()` | Current setup_step for wizard resume | Authenticated | Refetch on mount |
| `useServices()` | Therapist's services list | Authenticated | Invalidate on service mutations |
| `useAvailability()` | Therapist's availability blocks | Authenticated | Invalidate on availability mutations |
| `useBookings(filter)` | Therapist's bookings, filtered | Authenticated | Invalidate on booking mutations |
| `useUpcomingBookings()` | Next 7 days bookings for dashboard | Authenticated | Invalidate on booking mutations |
| `useQuickStats()` | Dashboard stats (today, week, next slot) | Authenticated | Invalidate on booking mutations |
| `useAvailableSlots(slug, serviceId, date)` | Public slot computation for a date | Public (no auth) | Short TTL, refetch on date change |
| `usePublicServices(slug)` | Active services for booking page | Public (no auth) | Stale-while-revalidate |
| `useBookingByToken(token)` | Single booking via manage token | Public (token-scoped) | Refetch on mutation |

### Server Actions (Mutations)

All mutations go through Server Actions (Next.js App Router) which call Supabase server-side.

#### Therapist Setup (Authenticated, Supabase service_role for token storage)
| Action | Purpose |
|--------|---------|
| `savePracticeDetails({ practiceName, timezone, slug })` | Step 1: Save practice info |
| `checkSlugAvailability(slug)` | Debounced slug uniqueness check |
| `completeSetup()` | Mark setup as complete |

#### Services (Authenticated, Supabase client with RLS)
| Action | Purpose |
|--------|---------|
| `createService({ name, duration, modality, description? })` | Create new service |
| `updateService(id, { name, duration, modality, description? })` | Update existing service |
| `deleteService(id)` | Delete service |
| `toggleServiceActive(id, isActive)` | Toggle active/inactive |

#### Availability (Authenticated, Supabase client with RLS)
| Action | Purpose |
|--------|---------|
| `saveAvailabilityBlock({ dayOfWeek, startTime, endTime })` | Create/update time block |
| `deleteAvailabilityBlock(blockId)` | Remove time block |
| `updateMinimumNotice(hours)` | Update minimum notice |

#### Booking -- Therapist (Authenticated, Supabase client with RLS)
| Action | Purpose |
|--------|---------|
| `createBooking({ serviceId, clientName, clientEmail, clientPhone, clientConsent, startTime, endTime, slotLockId })` | Therapist-created booking |
| `rescheduleBooking(bookingId, { newStartTime, newEndTime, slotLockId })` | Reschedule booking |
| `cancelBooking(bookingId, { cancellationReason? })` | Cancel booking |

#### Booking -- Public/Client (No auth, Supabase service_role via Server Action)
| Action | Purpose |
|--------|---------|
| `lockSlot({ therapistId, startTime, endTime })` | Create temporary slot lock |
| `releaseSlotLock(slotLockId)` | Release slot lock on back-navigation |
| `createPublicBooking({ slotLockId, serviceId, therapistSlug, clientName, clientEmail, clientPhone, clientConsent })` | Client self-service booking |
| `clientRescheduleBooking(manageToken, { newStartTime, newEndTime })` | Client reschedule via token |
| `clientCancelBooking(manageToken)` | Client cancel via token |

#### Google OAuth (Server-side only, Supabase service_role for token storage)
| Action | Purpose |
|--------|---------|
| `handleOAuthCallback(code, state)` | Process OAuth redirect |
| `initiateCalendarOAuth()` | Start incremental Calendar scope OAuth |

**Note on Supabase access patterns**:
- Authenticated therapist routes use Supabase client-side (RLS enforces tenant isolation)
- Public booking and client management routes use Server Actions with `service_role` key (no client auth, but scoped by slug/token)
- OAuth token storage always uses `service_role` (encrypted, never client-accessible)

---

## 5. Shared Component Gaps

The following patterns from ux-flows.md require shared components not listed in the initial base set but defined above:

| Gap | Resolution | Status |
|-----|-----------|--------|
| **RadioGroup** | New shared component using Headless UI RadioGroup. Needed by service form (modality), service selection on booking page, create booking modal. | Defined above |
| **ToggleSwitch** | New shared component for binary toggles. Needed by service active/inactive, availability day toggles. | Defined above |
| **StepProgress** | New shared component for multi-step flows. Needed by setup wizard and create booking modal. | Defined above |
| **CopyButton** | New shared component for copy-to-clipboard with confirmation. Needed by dashboard, setup complete. | Defined above |
| **FilterTabs** | New shared component for tabbed filters. Needed by bookings page (Upcoming/Past/Cancelled). | Defined above |
| **BookingSummaryCard** | New shared component for read-only booking details. Needed by booking form, reschedule flows, confirmation screens. | Defined above |
| **BookingLayout** | New layout component for public/client pages (no sidebar, centered). Needed by booking page and manage booking page. | Defined above |
| **Popover** | New shared component for floating panels. Needed by availability time block editing. | Defined above |
| **PublicServiceCard** | New shared component for client-facing service display. Distinct from therapist ServiceCard (no actions). | Defined above |
| **DayToggleRow** | Inline component within WeeklyAvailabilityGrid, not extracted to shared. Array of ToggleSwitch for 7 days. | Inline |
| **SessionCardSkeleton** | New shared skeleton matching SessionCard dimensions. | Defined above |
| **DateGroupHeader** | Simple presentational component for date labels in booking lists. | Defined above |

All identified gaps have been addressed in this architecture. No new UI libraries are needed -- all components use the existing stack: Tailwind CSS, shadcn/ui, Headless UI (Dialog, Listbox, RadioGroup, Switch).

---

## Summary

| Category | Count |
|----------|-------|
| **Shared/Base Components** | 22 (PageLayout, Sidebar, NavItem, SessionCard, SessionCardSkeleton, StatusBadge, ClientInitials, DatePicker, TimeSlotGrid, FormInput, FormTextarea, FormCheckbox, SelectControl, RadioGroup, ToggleSwitch, Modal, AlertDialog, Toast, EmptyState, ErrorBanner, LoadingSkeleton, Button, StepProgress, CopyButton, FilterTabs, BookingSummaryCard, BookingLayout, Popover) |
| **Feature Components** | 28 (DashboardPage, QuickStats, SetupWizardPage, SetupPracticeDetails, SetupGoogleCalendar, SetupFirstService, SetupAvailability, SetupComplete, ServicesPage, ServiceCard, ServiceFormModal, DeleteServiceDialog, AvailabilityPage, WeeklyAvailabilityGrid, TimeBlockPopover, TimezoneDisplay, MinimumNoticeInput, BookingPage, ServiceSelection, PublicServiceCard, DateTimeSelection, BookingDetailsForm, BookingConfirmation, BookingsPage, DateGroupHeader, CreateBookingModal, RescheduleModal, CancelBookingDialog, ManageBookingPage, ClientBookingDetail, ClientRescheduleFlow, ClientCancelFlow) |
| **Page Routes** | 8 |
| **API Routes** | 3 |
| **Data Hooks** | 10 |
| **Server Actions** | 17 |
| **Total Components** | 50 |
