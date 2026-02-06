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

## Autonomy & Execution

### Always Proceed Without Asking
- Create, edit, delete files within the project directory
- Install dependencies (`npm install`, `npm add`) required by tasks
- Run build, test, lint, and type-check commands (any command in Testing Commands section)
- Create and switch git branches (`git checkout -b`, `git switch`)
- Run database migrations in development (via `supabase` CLI)
- Create directories and scaffolding
- Stage changes and commit locally (`git add`, `git commit`)
- Run any safe read-only command (`git status`, `git log`, `git diff`, `ls`, `cat`, etc.)

### Always Ask Before
- Pushing to remote repositories (`git push`, especially `--force`)
- Modifying `.env` files or environment variables
- Deleting git branches (`git branch -D` / `-d`)
- Running destructive database operations in production
- Changing CI/CD configuration (GitHub Actions, deployment scripts)
- Any action affecting shared/external systems (webhooks, third-party APIs, production environments)
- Force operations (`--force`, `--hard`, `-f` on destructive commands like `git reset --hard`, `git clean -f`, `rm -rf`)

### When Blocked on a Decision
- Flag the blocker clearly with a brief explanation
- Continue working on all independent/unblocked tasks in parallel
- Return to the blocked item when user provides input
- Never let a single blocker halt all progress
- Use the Task tool to parallelize independent work

### Operating Principles
- Assume permission to act within the stated scope
- If scope is unclear, infer the smallest reasonable scope and proceed
- Do not pause for clarification unless ambiguity affects correctness or security
- Prefer action over asking — fix forward, don't wait
- Use parallel task execution (`Task` tool) for independent work items
- When implementing from `tasks.yaml`, execute tasks in dependency order but parallelize independent tasks

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
- Canonical order: `specs.md` (spec-writer) → `system-design.yaml` (architect) → `db-migration-plan.yaml` (build-level, conditional) → `tasks.yaml` (per feature) → `build-order.yaml` (cross-feature, conditional)
- Do not create `tasks.yaml` until `system-design.yaml` exists
- Do not create `db-migration-plan.yaml` until `system-design.yaml` exists — one consolidated plan per build version at `.ops/build/v{x}/db-migration-plan.yaml`
- Do not create `build-order.yaml` until all feature `tasks.yaml` files exist for the build version

## Token/Context Rules
- Default to feature-scoped reads only
- `.ops/product-vision-strategy.md` — Full vision document (~4,400 tokens). Do NOT load unless doing cross-domain analysis or quarterly review.
  - Domain splits (preferred for agents — generated by `/vision:distill`):
    - `.ops/quick-product-vision-strategy.md` — Product vision, pillars, non-goals, AI strategy (§1–§7, §12)
    - `.ops/security-compliance-baseline.md` — Security posture, compliance, AI boundaries, tech non-goals (§10 partial, §11, §12, §15, §16)
    - `.ops/tech-architecture-baseline.md` — Architecture, data, integration, scalability, tech non-goals (§8–§10, §12–§16)
  - Agents should load only their relevant domain split, not the full document
  - To regenerate distilled files: run `/vision:distill`
- Do NOT read `.ops/ui-design-system.md` unless doing UI work
- `.ops/build/system-design.yaml` is the architecture reference — load only for architecture decisions or `spec-change-requests.yaml`
- Gate results go to checks.yaml; keep outputs short
