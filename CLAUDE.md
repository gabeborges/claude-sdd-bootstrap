# Security, Identity, & Baseline Defaults

This defines **absolute guards** that must be followed for all Claude Code and agent operations across any repo or project.

---

## NEVER EVER DO — GATEKEEPER RULES (Absolute)

### 🚫 NEVER Publish Sensitive Data
- Never publish passwords, API keys, tokens, or credentials in code, documentation, comments, PRs, branch names, commits, issue titles, diffs, or logs.
- Before ANY commit, ensure no secrets exist in staged changes.

### 🔒 NEVER Commit `.env` or Credential Files
- `.env`, `.env.local`, `.env.*`, credential files, secret files, and config files containing secrets must NEVER be committed.
- Confirm `.gitignore` properly excludes these before code generation.

### ❌ NEVER Hardcode Credentials
- Always use secure environment variables or secret stores (e.g., Vercel environment settings, Supabase secrets) for all credentials.
- Do not embed API keys, secret tokens, database credentials, OAuth secret values, or similar in code or configuration.

---

## Project Defaults (Global)

### Identity
- Default GitHub identity:
  - `git config user.name = YourGitHubUsername`
  - `git config user.email = your.email@example.com`
- Default repository origin format for CI/CD:
  - `git@github.com:YourGitHubUsername/<repo>.git`

### Repository Baseline
- Required at project creation:
  - `.gitignore` covering all secret files
  - `.env.example` with placeholder values (no actual secrets)
  - `README.md` with environment setup instructions

---

## Enforced Tool Use

### PreToolUse / Guard Hooks (Highly Recommended)
- Strategies (optional but recommended):
  - PreToolUse hook that rejects edits to any `*.env*` file
  - PreToolUse hook that rejects commits with secret patterns (`KEY=`, private tokens)
- These hooks run before Claude Code or local tools execute.

---

## Safety & Model Constraints

### Output Constraints
- The model MUST NOT output secrets, even if they exist locally.
- If a suggestion would include sensitive data, it MUST issue a warning like:
  > “Proposal contains sensitive data — remove before proceeding.”

### Fault Reporting
- If illegal/sensitive output is generated, stop and log the occurrence into:
  - `.ops/audit/clause/security_incidents.log`

---

## Security Priority
These rules are **non-negotiable** and apply across all repositories, agents, and use cases.

---

END OF GLOBAL CLAUDE.md

---

# Execution Guardrails (SDD + Next.js + Stack)

This defines **project-specific execution constraints** for Claude Code and agent execution.

Stack: Next.js (App Router), Tailwind, shadcn/ui, Headless UI, Supabase, Stripe, Google OAuth  
Testing: Vitest + Playwright  
Package manager: npm  
Development workflow: SDD (Clavix + Agent Swarm)

---

## 0) SDD First (Mandatory)

Before writing or changing code, you MUST read:
- `.ops/product-vision-strategy.md`
- `.ops/build/v{x}/prd.md`
- `.ops/build/system-design.yaml`
- `.ops/build/v{x}/<feature-name>/specs.md`
- `.ops/build/v{x}/<feature-name>/tasks.md`

**Do not proceed with coding if any artifact above is missing or ambiguous. Ask questions.**

---

## 1) Workflow Discipline

### Spec vs Implementation
- Write `specs.md` and `implementation-status.md` only in the proposal phase.
- Only implement code from tasks in `tasks.md` (feature-level).

### File Roles
- `implementation-status.md` → version-level progess status in `.ops/build/v{x}/`
- `specs.md` → requirements + acceptance criteria in `.ops/build/v{x}/<feature-name>/specs.md`
- `tasks.md` → feature tickets in `.ops/build/v{x}/<feature-name>/tasks.md` with `implements:` pointers into `specs.md`

---

## 2) Directory & Version Safety

- Work within `.ops/build/v{x}/<feature-name>/`
- Do not mix build versions (`v0`, `v1`, `v2`) in a single change or commit.

---

## 3) Next.js (App Router) Norms

### App Router Enforcement
- Stick to App Router conventions.
- Keep components modular and colocated with routes or features.

### Client vs Server
- Prefer **Server Components** by default.
- Use `"use client"` only where required by UI behavior.

---

## 4) Styling/UI Rules

- Use Tailwind for utility-style consistency.
- Use components from shadcn/ui.
- Use Headless UI for accessible primitives.

Do not add new UI libs without explicit approval or task references.

- If `.ops/ui-design-system.md` exists, UI work MUST comply with it.
- If UI work is needed and the system file is missing, run `/interface-design:init` before designing/implementing UI.
- For simple one-off pages/components where a project-wide system is not needed, prefer the `frontend-design` skill.

---

## 5) Data / Auth / Payments

### Supabase
- Respect existing RLS and permission boundaries.
- Sensitive logic goes server-side.

### Google OAuth
- Do not relax OAuth scopes, callback URLs, or token handling without spec reference.

### Stripe
- Respect existing semantics for billing, products, prices, and webhooks.
- Always verify webhook signatures.

---

## 6) Security & Secrets (Project Level)

- Do not log secrets.
- Do not commit credentials.
- Do not embed secret values in code or config.
- If spec conflicts with secure implementation:
  - Create a `spec-change-requests.md` entry and stop.

---

## 7) Testing & Validation

Before marking a task done:
- Validate code satisfies acceptance scenarios in `specs.md`.
- Add tests consistent with:
  - **Vitest** for logic and API tests
  - **Playwright** for E2E flows

Run project scripts (inspect `package.json`):
- `npm run lint`
- `npm run test`
- `npm run test:e2e`
- `npm run build`

---

## 8) Change Discipline (Minimal Diff)

- Implement the *smallest change* that satisfies the task.
- Do not refactor unrelated code.
- Avoid renaming files unless required by tasks.

---

## 9) Output Requirements

When completing a ticket:
- Describe what puzzles/scenarios it satisfies (`implements:` pointers).
- Show tests run and results.
- Attach reproducible steps or validation notes.

---

## 10) Stop & Ask Conditions

Stop and ask:
- If build version `v{x}` isn’t clear
- If required spec artifacts are missing
- If tasks lack `implements:` pointers
- If a secret situation/security risk is unclear

---

END OF PROJECT CLAUDE.md

## Token/Context Rules
- Default to feature-scoped reads only.
- Do NOT read `.ops/product-vision-strategy.md` or `.ops/ui-design-system.md` unless needed.
- Gate results go to checks.yaml; keep outputs short.


## System Design Artifact
- `./ops/build/system-design.yaml` is the evolving architecture reference for the whole product.
- Do NOT load it unless working on architecture decisions or resolving `spec_change_requests`.
