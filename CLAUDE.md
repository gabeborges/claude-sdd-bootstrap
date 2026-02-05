# Project Rules (Auto-loaded Every Session)

## Stack
Next.js (App Router), Tailwind, shadcn/ui, Headless UI, Supabase, Stripe, Google OAuth
Testing: Vitest + Playwright | Package manager: npm | Workflow: SDD (Clavix + Agent Swarm)

## Security (Non-negotiable)

- NEVER publish passwords, API keys, tokens, or credentials in code, docs, commits, PRs, diffs, or logs
- NEVER commit `.env`, `.env.*`, credential files, or config files containing secrets — verify `.gitignore` covers them
- NEVER hardcode credentials — use environment variables or secret stores
- NEVER output secrets even if they exist locally — warn: "Proposal contains sensitive data — remove before proceeding."
- NEVER log PHI/PII payloads — log only request IDs, timestamps, non-sensitive error codes
- NEVER weaken auth checks, permission gates, or move privileged logic to the client
- Supabase RLS is non-negotiable — never bypass it
- Before ANY commit, verify no secrets in staged changes
- If security conflicts with spec: create `spec-change-requests.yaml` entry and stop

## Git Identity
- `user.name = YourGitHubUsername`
- `user.email = your.email@example.com`
- Origin: `git@github.com:YourGitHubUsername/<repo>.git`

## Next.js (App Router)
- App Router (`app/`) is authoritative
- Prefer Server Components by default; use `"use client"` only when required
- Server Actions: only for mutations
- Keep components modular and colocated with routes or features

## Styling / UI
- Tailwind for utility styles, shadcn/ui for components, Headless UI for accessible primitives
- No new UI libs without explicit approval or task reference
- If `.ops/ui-design-system.md` exists, UI work MUST comply with it
- If UI work needed and system file missing, run `/interface-design:init` first
- For simple one-off pages/components, prefer the `frontend-design` skill

## Architecture Boundaries

### Feature Isolation
- No cross-feature imports — shared code goes in `lib/`, `components/`, `utils/`

### Data Access (Supabase)
- UI must not call Supabase directly — use Edge Functions or server-side boundaries

### Stripe
- Server/client config: `/lib/stripe.js`
- Checkout: `/app/api/checkout/route.js`
- Frontend: `@stripe/react-stripe-js`
- Always verify webhook signatures
- Do not alter billing semantics, price IDs, or webhook handling without spec/tasks

### Google OAuth
- Do not relax OAuth scopes, callback URLs, or token handling without spec reference

## Coding Standards
- React components: **PascalCase**
- Files/folders: **kebab-case** (unless Next.js routing requires otherwise)
- APIs/routes: REST + Next.js App Router conventions
- ESLint (`next/core-web-vitals`) + Prettier required
- Do not disable lint rules without task/spec justification documented in `.ops/build/decisions-log.md`
- Minimal diff: smallest change that satisfies the ticket
- No unrelated refactors, renames, or formatting-only churn
- If a shared module is touched, add/adjust tests proportional to risk

## Testing Commands
```
npm run lint
npm run test
npm run test:e2e
npm run build
```

## Version Safety
- Work within `.ops/build/v{x}/<feature-name>/`
- Do not mix build versions (`v0`, `v1`, `v2`) in a single change or commit

## SDD Artifact Flow
- Canonical order: `specs.md` (spec-writer) → `system-design.yaml` (architect) → `tasks.yaml` (project-task-planner)
- Do not create `tasks.yaml` until `system-design.yaml` exists

## Token/Context Rules
- Default to feature-scoped reads only
- Do NOT read `.ops/product-vision-strategy.md` or `.ops/ui-design-system.md` unless needed
- `.ops/build/system-design.yaml` is the architecture reference — load only for architecture decisions or `spec_change_requests`
- Gate results go to checks.yaml; keep outputs short
