# Curio UI Design System

**Product:** Practice management for solo online therapists
**Direction:** Warm professional notebook meets modern calendar tool
**Feel:** Trustworthy, calm, approachable — not clinical, not playful

---

## Intent & Audience

**Who:** Solo online therapists working from home between sessions. They're managing client care, documentation, and admin. They need tools that feel calm, reduce cognitive load, and signal trustworthiness without technical jargon.

**Feel:** A well-organized practice journal that happens to be digital. Warm enough to feel human, professional enough to handle sensitive clinical work, simple enough to not add stress.

---

## Core Direction

### Domain Concepts
Clinical warmth, therapeutic calm, practice documentation, Google Workspace integration, compliance-first safety, therapist-client trust, professional notebooks

### Color World
Quality paper (cream/beige), soft charcoal ink, therapeutic greens (muted sage), calm blues (soft teal), warm neutrals, document whites

### Signature Element
**Trust indicator** — A subtle, always-visible signal that data is protected. Not a technical badge or lock icon. More like a warm glow or soft indicator that says "safe space" without shouting about security. Appears in contexts where therapists handle client data.

### Rejected Defaults
1. **Generic SaaS dashboard** → Warm, document-like surfaces
2. **Healthcare EHR sterility** → Approachable warmth
3. **Heavy admin panels** → Calm, focused interfaces

---

## Foundation Tokens

### Color Palette

```css
/* Base surfaces - warm cream/beige (like quality paper) */
--surface-base: hsl(40, 20%, 98%);        /* Main canvas - warm off-white */
--surface-100: hsl(40, 22%, 99%);         /* Elevated cards - slightly lighter */
--surface-200: hsl(40, 25%, 99.5%);       /* Floating elements - highest elevation */

/* Alternative surfaces - for inset/recessed areas */
--surface-inset: hsl(40, 18%, 96%);       /* Slightly darker for depth */

/* Text hierarchy - soft charcoal (not harsh black) */
--text-primary: hsl(40, 8%, 18%);         /* Main content - warm dark gray */
--text-secondary: hsl(40, 6%, 40%);       /* Supporting text */
--text-tertiary: hsl(40, 5%, 55%);        /* Metadata, timestamps */
--text-muted: hsl(40, 4%, 70%);           /* Disabled, placeholder */

/* Borders - subtle, warm-tinted */
--border-default: hsl(40, 15%, 88%);      /* Standard borders */
--border-subtle: hsl(40, 12%, 92%);       /* Softer separation */
--border-strong: hsl(40, 18%, 78%);       /* Emphasis, hover */
--border-stronger: hsl(40, 20%, 65%);     /* Focus rings */

/* Brand accent - muted sage (calm, therapeutic, distinct) */
--accent: hsl(150, 25%, 45%);             /* Primary accent - muted sage */
--accent-hover: hsl(150, 27%, 40%);       /* Hover state */
--accent-active: hsl(150, 30%, 35%);      /* Active/pressed state */
--accent-subtle: hsl(150, 20%, 95%);      /* Backgrounds, badges */

/* Semantic colors - warm-adjusted */
--semantic-success: hsl(140, 35%, 42%);   /* Success actions */
--semantic-warning: hsl(38, 55%, 50%);    /* Warnings, caution */
--semantic-error: hsl(0, 50%, 48%);       /* Errors, destructive */
--semantic-info: hsl(200, 45%, 48%);      /* Info, neutral callouts */

/* Success/warning/error subtle backgrounds */
--semantic-success-bg: hsl(140, 30%, 96%);
--semantic-warning-bg: hsl(38, 45%, 96%);
--semantic-error-bg: hsl(0, 40%, 96%);
--semantic-info-bg: hsl(200, 35%, 96%);
```

### Typography

```css
/* Font families */
--font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--font-mono: 'JetBrains Mono', 'Fira Code', 'SF Mono', Consolas, monospace;

/* Type scale */
--text-xs: 0.75rem;      /* 12px - tiny labels */
--text-sm: 0.875rem;     /* 14px - UI text, labels */
--text-base: 1rem;       /* 16px - body text */
--text-lg: 1.125rem;     /* 18px - emphasized text */
--text-xl: 1.25rem;      /* 20px - section headings */
--text-2xl: 1.5rem;      /* 24px - page headings */
--text-3xl: 1.875rem;    /* 30px - hero text */

/* Font weights */
--font-normal: 400;
--font-medium: 500;
--font-semibold: 600;

/* Line heights */
--leading-tight: 1.25;   /* Headings */
--leading-normal: 1.5;   /* Body text */
--leading-relaxed: 1.75; /* Spacious reading */

/* Letter spacing */
--tracking-tight: -0.01em;   /* Headings */
--tracking-normal: 0;         /* Body */
--tracking-wide: 0.02em;      /* Small caps, labels */
```

### Spacing Scale

```css
/* Base unit: 8px */
--space-1: 0.25rem;   /* 4px - micro gaps */
--space-2: 0.5rem;    /* 8px - tight spacing */
--space-3: 0.75rem;   /* 12px - comfortable gaps */
--space-4: 1rem;      /* 16px - standard spacing */
--space-5: 1.25rem;   /* 20px - section spacing */
--space-6: 1.5rem;    /* 24px - larger gaps */
--space-8: 2rem;      /* 32px - major separation */
--space-10: 2.5rem;   /* 40px - section breaks */
--space-12: 3rem;     /* 48px - page-level spacing */
--space-16: 4rem;     /* 64px - dramatic separation */
```

### Border Radius

```css
--radius-sm: 0.25rem;   /* 4px - inputs, small buttons */
--radius-md: 0.375rem;  /* 6px - buttons, controls */
--radius-lg: 0.5rem;    /* 8px - cards, panels */
--radius-xl: 0.75rem;   /* 12px - modals, large containers */
--radius-full: 9999px;  /* Pills, circular avatars */
```

### Shadows & Depth

**Strategy:** Subtle borders + minimal shadows. Borders define structure, very soft shadows on floating elements to maintain warmth.

```css
/* Soft shadows - barely there, warm-tinted */
--shadow-sm: 0 1px 2px hsl(40 15% 50% / 0.04);
--shadow-md: 0 2px 4px hsl(40 15% 50% / 0.05),
             0 1px 2px hsl(40 15% 50% / 0.03);
--shadow-lg: 0 4px 8px hsl(40 15% 50% / 0.06),
             0 2px 4px hsl(40 15% 50% / 0.04);

/* Focus ring - for accessibility */
--shadow-focus: 0 0 0 3px var(--accent-subtle),
                0 0 0 4px var(--border-stronger);
```

---

## Component Patterns

### Buttons

**Primary button** (main actions - e.g., "Sign in with Google"):
```css
background: var(--accent);
color: white;
padding: var(--space-3) var(--space-5);
border-radius: var(--radius-md);
font-weight: var(--font-medium);
transition: background 150ms ease-out;

/* Hover */
background: var(--accent-hover);

/* Active */
background: var(--accent-active);

/* Focus */
box-shadow: var(--shadow-focus);
```

**Secondary button** (less emphasis):
```css
background: transparent;
color: var(--text-primary);
border: 1px solid var(--border-default);
padding: var(--space-3) var(--space-5);
border-radius: var(--radius-md);
font-weight: var(--font-medium);

/* Hover */
background: var(--surface-inset);
border-color: var(--border-strong);
```

### Cards

**Standard card**:
```css
background: var(--surface-100);
border: 1px solid var(--border-subtle);
border-radius: var(--radius-lg);
padding: var(--space-6);
box-shadow: var(--shadow-sm);
```

### Inputs

**Text input**:
```css
background: var(--surface-inset);
border: 1px solid var(--border-default);
border-radius: var(--radius-sm);
padding: var(--space-3) var(--space-4);
color: var(--text-primary);
font-size: var(--text-base);

/* Focus */
border-color: var(--border-stronger);
box-shadow: var(--shadow-focus);
outline: none;

/* Error */
border-color: var(--semantic-error);
```

### Navigation

**Sidebar** (when applicable):
```css
background: var(--surface-base);  /* Same as main canvas */
border-right: 1px solid var(--border-subtle);
padding: var(--space-6);
```

### Loading States

**Spinner/loader**:
```css
color: var(--accent);
/* Use subtle animation, ~150ms ease-out */
```

**Skeleton loading**:
```css
background: var(--surface-inset);
border-radius: var(--radius-sm);
/* Subtle pulse animation */
```

---

## Auth-Specific Patterns

### Login Page

**Layout:**
- Centered card on warm base surface
- Minimal chrome, focused content
- Optional subtle background pattern (warm paper texture)

**Card structure:**
```css
background: var(--surface-100);
border: 1px solid var(--border-subtle);
border-radius: var(--radius-xl);
padding: var(--space-10);
box-shadow: var(--shadow-md);
max-width: 400px;
margin: 0 auto;
```

**Content hierarchy:**
1. Logo or product name (text-2xl, semibold, text-primary)
2. Welcome message (text-base, text-secondary)
3. "Sign in with Google" button (primary button, full width)
4. Error message (if needed) - semantic-error color, error-bg background

### Error States

**Insufficient scopes error:**
```css
background: var(--semantic-error-bg);
border-left: 3px solid var(--semantic-error);
padding: var(--space-4);
border-radius: var(--radius-md);
color: var(--text-primary);

/* Icon */
color: var(--semantic-error);

/* Heading */
font-weight: var(--font-semibold);
color: var(--semantic-error);

/* Body text */
color: var(--text-secondary);
```

### Loading States (OAuth redirect)

**Loading indicator:**
```css
/* Centered spinner with message */
color: var(--accent);
text-align: center;

/* Message */
font-size: var(--text-sm);
color: var(--text-tertiary);
margin-top: var(--space-4);
```

---

## Accessibility

### Focus States
All interactive elements must have visible focus indicators using `--shadow-focus`.

### Color Contrast
- Text on `--surface-base`: Minimum WCAG AA contrast (4.5:1 for body, 3:1 for large text)
- All semantic colors tested against their background variants

### Keyboard Navigation
All interactive elements must be reachable and operable via keyboard.

---

## Usage Guidelines

### When to Use This System
- All therapist-facing interfaces (dashboard, settings, admin)
- Auth flows (login, logout, session management)
- Clinical documentation interfaces
- Booking/scheduling UIs

### When NOT to Use This System
- Public-facing marketing pages (may have different tone)
- Client-facing booking page (may need additional warmth/approachability)

### Extending the System
New components should:
1. Use only foundation tokens (no arbitrary values)
2. Follow the subtle layering principle (barely different surfaces, light borders)
3. Match the warm, professional notebook feel
4. Be accessible (focus states, contrast, keyboard nav)

---

## Technical Implementation

### CSS Variables
Define all tokens as CSS custom properties at `:root` level.

### Tailwind Integration
Map tokens to Tailwind config:
```js
// tailwind.config.js
theme: {
  extend: {
    colors: {
      'surface-base': 'hsl(40, 20%, 98%)',
      'accent': 'hsl(150, 25%, 45%)',
      // ... map all tokens
    },
    spacing: {
      // Map --space-* to Tailwind spacing scale
    },
  },
}
```

### shadcn/ui Components
Use shadcn/ui as base, override with Curio tokens for:
- Colors (surfaces, text, borders, accents)
- Spacing (padding, gaps)
- Border radius
- Shadows

---

## Checks Before Shipping

1. **Squint test:** Can you perceive hierarchy without harsh lines?
2. **Warmth test:** Does it feel warm/approachable, not clinical?
3. **Token test:** Are all values from the system (no arbitrary hex codes)?
4. **Accessibility test:** Focus states visible, contrast ratios meet WCAG AA?
5. **Signature test:** Does the trust indicator appear where needed?

---

*Design system created: 2026-02-03*
*For: Curio auth feature (login, session management)*
*Principle: Warm professional notebook meets modern calendar tool*
