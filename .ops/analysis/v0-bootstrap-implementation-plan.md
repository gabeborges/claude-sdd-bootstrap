# V0 Bootstrap System — Implementation Plan

**Date:** 2026-02-06
**Source Prompt:** `.clavix/outputs/prompts/comp-20260206-121500-v0ag.md`
**Perspective:** Claude Code Specialist (agent/skill/command architecture)
**Status:** Proposal — awaiting approval

---

## Executive Summary

The v0 bootstrap system requires **3 new artifacts** (1 command, 1 skill, 1 agent), **2 modifications** to existing files, and a **stack-profile convention** that lives alongside the existing SDD workflow. The system is designed to be stack-agnostic — it reads what a project uses from `CLAUDE.md` and generates the right scaffolding.

---

## Current State

| Asset | Exists? | Notes |
|-------|---------|-------|
| Bootstrap agent | No | Nothing in the 21-agent roster handles initialization |
| Bootstrap command | No | None of the 21 commands trigger project setup |
| Bootstrap skill | No | None of the 8 skills cover env scaffolding or stack discovery |
| `.ops/build/v0/` | No | No build versions exist yet |
| `.ops/.v0-initialized` | No | No sentinel file |
| `.env.local.example` | No | No env template in the repo |
| Stack declaration | Yes | `CLAUDE.md` line 3: `Next.js (App Router), Tailwind, shadcn/ui, Headless UI, Supabase, Stripe, Google OAuth` |

---

## Implementation Plan

### Phase 1: Create the Bootstrap Command (Entry Point)

**File:** `.claude/commands/bootstrap/init.md`

**Why a command (not just an agent)?**
- Commands are user-invocable (`/bootstrap:init`) — gives explicit control
- Commands can also be triggered programmatically by the workflow-orchestrator
- Keeps the bootstrap logic visible in the command roster

**Responsibilities:**
1. Check sentinel `.ops/.v0-initialized` — exit early if present
2. Call stack discovery (Phase 2 skill) to parse `CLAUDE.md ## Stack`
3. Execute profile-specific scaffolding steps
4. Generate the user dependency checklist
5. Write sentinel file with timestamp + stack profile

**Frontmatter:**
```yaml
---
name: Initialize Project (v0 Bootstrap)
description: One-time project setup — detects stack from CLAUDE.md, scaffolds env, seeds, tooling, and checklist. Runs once.
model: sonnet
tools: Read, Write, Edit, Glob, Grep, Bash
---
```

**Suggestion:** Keep the command under 200 lines. It should be a clear sequential workflow:
1. Guard (sentinel check)
2. Discover (read stack)
3. Scaffold (env, gitignore, configs)
4. Seed (generate data fixtures)
5. Checklist (user-facing instructions)
6. Finalize (sentinel + summary)

Each step should be a clearly-marked section so the agent can report progress.

---

### Phase 2: Create the Stack Discovery Skill

**File:** `.claude/skills/stack-discovery/SKILL.md`

**Why a skill (not inline in the command)?**
- Stack discovery is reusable — other agents/commands may need it later (e.g., `spec-writer` inferring tech constraints, `architect` validating system-design.yaml)
- Skills are loaded on-demand, keeping the command lean
- The knowledge base (service → env vars mapping) can grow independently

**Responsibilities:**
1. Read `CLAUDE.md` → extract `## Stack` section
2. Parse into structured profile: `{ platform, framework, packageManager, database, auth, payments, services[] }`
3. For each recognized service, provide:
   - Required env vars (names + descriptions)
   - Config file templates needed
   - `.gitignore` patterns
   - Seed data format
   - Checklist steps (signup URL, what to get, where to paste)

**Known Service Mappings (initial set):**

| Service | Env Vars | Config Files | Seed Format |
|---------|----------|-------------|-------------|
| Supabase | `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` | — | `supabase/seed.sql` |
| Stripe | `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`, `STRIPE_WEBHOOK_SECRET` | — | `stripe/fixtures.json` |
| Google OAuth | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` | — | — |
| NextAuth | `NEXTAUTH_SECRET`, `NEXTAUTH_URL` | — | — |
| Firebase | `FIREBASE_API_KEY`, `FIREBASE_AUTH_DOMAIN`, `FIREBASE_PROJECT_ID`, `FIREBASE_STORAGE_BUCKET` | `firebase.json`, `.firebaserc` | `firebase/seed.json` |
| Vercel | `VERCEL_TOKEN` | `vercel.json` | — |
| Next.js | — | `next.config.js`, `tsconfig.json` | — |
| Tailwind | — | `tailwind.config.ts`, `postcss.config.js` | — |

**Suggestion:** Store the mapping knowledge in a references file (`.claude/skills/stack-discovery/references/service-profiles.md`) to keep the SKILL.md lean. The skill body describes the *process*, references hold the *data*.

**Fallback behavior:** If a technology in `## Stack` isn't in the known mappings, the skill should:
1. Note it as "unrecognized" in the output
2. Still proceed with known services
3. Add a manual step to the checklist: "Configure [unknown-service] — check their docs for required env vars"

---

### Phase 3: Modify Existing Files

#### 3a. Update `CLAUDE.md` — Add Bootstrap Section

Add a short section (4-5 lines) after `## SDD Artifact Flow`:

```markdown
## V0 Bootstrap
- First-time project setup: run `/bootstrap:init` or let workflow-orchestrator auto-detect
- Sentinel: `.ops/.v0-initialized` — if present, bootstrap is complete
- Bootstrap reads `## Stack` above and generates env templates, seeds, and checklist
- Do NOT manually create `.ops/build/v0/` — bootstrap handles this
```

**Why in CLAUDE.md?** Every agent sees this. It prevents agents from:
- Manually creating v0 structures that conflict with bootstrap output
- Re-running bootstrap logic that already completed
- Skipping bootstrap when it hasn't run yet

**Token cost:** ~4 lines = ~40 tokens. Acceptable.

#### 3b. Update `claude/agents/workflow-orchestrator.md` — Add V0 Awareness (Informational)

Add a lightweight note to the orchestrator's workflow section:

```markdown
### V0 Awareness (Informational Only)
- If `.ops/.v0-initialized` does NOT exist, note in status output:
  "Project has not been bootstrapped. Run /bootstrap:init when ready."
- Do NOT block, prompt, or auto-trigger bootstrap.
```

**Why informational only?** User decided bootstrap should be explicitly invoked. The orchestrator just surfaces awareness — 3 lines, no logic branching.

#### 3c. Update `AGENTS.md` — Add Bootstrap to Agent Roster

Add to the agent roster table and dependency DAG:

- **Tier 0 (new):** `bootstrap:init` — runs before any other SDD tier, but only once
- Add to the "By Category" section under a new `setup` category

---

### Phase 4: Create Sentinel and Output Structure

When bootstrap runs, it creates:

```
.ops/
├── .v0-initialized              # Sentinel (YAML: timestamp, stack_profile, version)
└── build/
    └── v0/
        └── setup-checklist.md   # User-facing dependency setup guide
```

**Sentinel file format:**
```yaml
initialized_at: 2026-02-06T12:00:00Z
stack_profile:
  platform: web
  framework: Next.js (App Router)
  package_manager: npm
  database: Supabase
  auth: Google OAuth, NextAuth
  payments: Stripe
  ui: Tailwind, shadcn/ui, Headless UI
bootstrap_version: 1.0.0
artifacts_created:
  - .env.local.example
  - .ops/build/v0/setup-checklist.md
```

**Why YAML?** Agents can parse it. The `stack_profile` section becomes a reusable reference that other agents can read without re-parsing `CLAUDE.md`.

---

## Artifact Summary

| # | Action | File Path | Type | Lines (est.) |
|---|--------|-----------|------|-------------|
| 1 | **Create** | `.claude/commands/bootstrap/init.md` | Command | ~150 |
| 2 | **Create** | `.claude/skills/stack-discovery/SKILL.md` | Skill | ~80 |
| 3 | **Create** | `.claude/skills/stack-discovery/references/service-profiles.md` | Reference | ~120 |
| 4 | **Edit** | `CLAUDE.md` | Project rules | +5 lines |
| 5 | **Edit** | `claude/agents/workflow-orchestrator.md` | Agent | +3 lines |
| 6 | **Edit** | `AGENTS.md` | Coordination | +5 lines |

**Total new files:** 3
**Total modified files:** 3
**Estimated total new content:** ~355 lines

---

## Architectural Suggestions

### 1. Command vs. Agent: Use a Command

The bootstrap system should be a **command** (`/bootstrap:init`), not a standalone agent.

**Rationale:**
- Agents are spawned as subprocesses with isolated context — they can't easily interact with the user for the checklist walkthrough
- Commands run in the main conversation context — they can show progress, ask questions, and guide the user through the checklist interactively
- The bootstrap logic is a linear workflow, not a role that needs its own identity

### 2. Skill Separation: Stack Discovery is Reusable

Extract stack discovery into a skill because:
- The `spec-writer` could reference it to auto-populate tech constraints in specs
- The `architect` could reference it to validate `system-design.yaml` against declared stack
- The `security-engineer` could reference it to know which service-specific security checks to apply
- Future commands (e.g., `stack:audit`, `deps:check`) would benefit from it

### 3. Don't Over-Engineer Service Profiles

The initial implementation should hardcode knowledge for the **6 services in the current CLAUDE.md** (Supabase, Stripe, Google OAuth, NextAuth, Next.js, Tailwind). Don't build a full plugin registry up front.

**Why:** The prompt envisions iOS/Firebase/React Native profiles, but those are future product types. Build for what exists today, extend when needed. The skill's reference file is easy to grow.

### 4. Checklist as Static Reference Document

The setup checklist (`.ops/build/v0/setup-checklist.md`) is a **static reference guide** — not an interactive walkthrough.

**Pattern:**
```markdown
## Step 1: Create Supabase Project
1. Go to https://supabase.com/dashboard
2. Click "New Project", name it, choose a region
3. Navigate to Settings → API
4. Copy **Project URL** → paste into `.env.local` as `NEXT_PUBLIC_SUPABASE_URL`
5. Copy **anon/public key** → paste as `NEXT_PUBLIC_SUPABASE_ANON_KEY`
6. Copy **service_role key** → paste as `SUPABASE_SERVICE_ROLE_KEY`
```

The command generates and saves the file, then tells the user where to find it. No step-by-step prompting.

### 5. Idempotency via Sentinel, Not File Checks

Use the sentinel file (`.ops/.v0-initialized`) as the single source of truth for "has bootstrap run?" — don't check for individual files (`.env.local.example`, `setup-checklist.md`) to determine state.

**Why:** Individual files might be deleted, moved, or edited by the user. The sentinel is the authoritative record.

### 6. `.clavix/config.json` — Unblock Implementation

The current `.clavix/config.json` has `planningOnly: true` and blocks `/clavix:implement`. To actually build this feature, that restriction will need to be lifted or the bootstrap command needs to be added to the allowed list.

### 7. Consider a `--force` Flag

For cases where:
- The user wants to re-run bootstrap after changing the stack
- Bootstrap partially failed and needs to retry
- The sentinel exists but artifacts are missing

The command should accept `$ARGUMENTS` that include `--force` to bypass the sentinel check.

---

## Implementation Order

```
Phase 1 ─── Create stack-discovery skill + references
              │
Phase 2 ─── Create bootstrap:init command (depends on skill)
              │
Phase 3 ─── Edit CLAUDE.md, workflow-orchestrator, AGENTS.md
              │
Phase 4 ─── Test: run /bootstrap:init on current project
              │
Phase 5 ─── Verify: sentinel created, env template correct,
             checklist matches declared stack
```

**Phases 1-3** can be done in a single `/clavix:implement` session.
**Phases 4-5** require the `planningOnly` restriction to be lifted.

---

## Resolved Decisions

| # | Question | Decision | Impact on Implementation |
|---|----------|----------|------------------------|
| 1 | Auto-run vs explicit trigger | **Explicit only** — `/bootstrap:init` | Orchestrator should NOT auto-trigger. Remove auto-detect from `workflow-orchestrator.md` edit scope. Orchestrator may *mention* bootstrap hasn't run (informational), but never triggers it. |
| 2 | Env template naming | **`.env.local.example`** | Hardcode this name in the bootstrap command. Stack-discovery skill maps all stacks to `.env.local.example` as the canonical template name. |
| 3 | Checklist verification | **Reference only — no verification** | Checklist is a static guide. The command generates it and shows it to the user. No connection tests, no step-by-step confirmation prompts. User reads and follows at their own pace. |
| 4 | Seed generation source | **From `system-design.yaml`** | Seeds are NOT hardcoded defaults. The bootstrap command reads `.ops/build/system-design.yaml` to discover table definitions, then generates `supabase/seed.sql` with matching schema + sample rows. If `system-design.yaml` doesn't exist yet, skip seeds and note it in the checklist. |

### Impact of Decision #1 on Plan

The original plan (Phase 3) proposed editing `workflow-orchestrator.md` to add V0 auto-detection. With the "explicit only" decision, this edit is **reduced to informational**:

```markdown
### V0 Awareness (Informational Only)
- If `.ops/.v0-initialized` does NOT exist, note in status output:
  "Project has not been bootstrapped. Run /bootstrap:init when ready."
- Do NOT block, prompt, or auto-trigger bootstrap.
```

This is lighter — ~3 lines instead of ~8.

### Impact of Decision #4 on Plan

The bootstrap command now has a **dependency on `system-design.yaml`** for seed generation. This changes the workflow:

```
/bootstrap:init
  ├── Stack Discovery (from CLAUDE.md)          ← always runs
  ├── Env Template (.env.local.example)         ← always runs
  ├── Tooling/Config (.gitignore, linters)      ← always runs
  ├── Seed Generation (from system-design.yaml) ← conditional
  │     └── If system-design.yaml missing: skip, add to checklist
  └── Checklist + Sentinel                      ← always runs
```

The stack-discovery skill does NOT need to handle seed generation. Instead, the bootstrap command reads `system-design.yaml` directly for schema definitions and generates seeds from that. This is a separate concern from stack discovery.

**New artifact consideration:** The seed generation logic may warrant its own section in the bootstrap command, or could reference the existing `db-migration` skill for schema-aware patterns.

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Stack parsing fails on unusual CLAUDE.md format | Medium | Medium | Fallback to manual mode; ask user to confirm stack |
| User skips checklist steps, app crashes | High | Low | Checklist has verification hints; not a hard gate |
| Sentinel exists but artifacts are corrupted | Low | Medium | `--force` flag to re-run; sentinel stores artifact list |
| New services added to stack after bootstrap | Medium | Low | Re-run with `--force` or manually edit env template |
| Token cost of stack-discovery skill | Low | Low | Skill is on-demand (~80 lines); only loaded when invoked |
