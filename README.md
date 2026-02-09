# claude-sdd-bootstrap

A complete SDD (Specification-Driven Development) toolkit for AI-powered product development with Claude Code and Cursor.

---

## What This Is

This is a **bootstrap/template project**, not a product. It provides a standardized SDD workflow, agent swarm, commands, skills, and guardrails that you copy into your real product repositories.

**Key Pattern: `claude/` → `.claude/`**

- `claude/` — Source templates in this repo (what you copy from)
- `.claude/` — Destination in your product repo (where you copy to)

---

## Quick Start

Get started in 3 steps:

1. **Install Clavix**:
   ```bash
   npm install -g clavix
   ```

2. **Copy to your product repo**:
   ```bash
   # In your product repo root:
   cp -r /path/to/claude-sdd-bootstrap/claude/ ./.claude/
   cp /path/to/claude-sdd-bootstrap/CLAUDE.md ./
   cp /path/to/claude-sdd-bootstrap/AGENTS.md ./
   cp /path/to/claude-sdd-bootstrap/.mcp.json ./
   ```

3. **Start building**:
   ```bash
   # In Claude Code or Cursor:
   /clavix:product                     # Create product vision
   /clavix:prd                         # Create version PRD
   /vision:distill                     # Split vision into agent-consumable domain files
   /orchestrate .ops/build/v0/         # Planning + specs for entire build
   /orchestrate .ops/build/v0/<feature-name>/  # Implement a specific feature
   ```

**Configure MCP**: Add env vars to `.env` (never commit). See [MCP Setup](#mcp-configuration).

---

## Tech Stack

Pre-configured for:

| Category | Tools |
|----------|-------|
| Framework | Next.js (App Router) |
| Styling | Tailwind, shadcn/ui, Headless UI |
| Backend | Supabase (auth + database) |
| Payments | Stripe |
| Auth | Google OAuth |
| Testing | Vitest + Playwright |
| Package Manager | npm |
| Workflow | SDD (Clavix + Agent Swarm) |

Adapt these defaults in `CLAUDE.md` for your stack.

---

## What's Included

### 1. Guardrails (Runtime Rules)

- **`CLAUDE.md`** — Auto-loaded every session. Defines stack, security rules, coding standards, architecture boundaries, and SDD artifact flow.
- **`AGENTS.md`** — Agent coordination reference. Defines artifact priority table, agent roles, and orchestration rules.

### 2. Agent Swarm (19 Specialized Agents)

| Category | Agents |
|----------|--------|
| **Planning & Architecture** | architect, spec-writer, project-task-planner |
| **Development** | fullstack-developer, ui-designer, database-administrator |
| **Quality** | code-reviewer, qa, test-automator, debugger |
| **Security & Compliance** | security-engineer, security-auditor, compliance-engineer, compliance-auditor |
| **Meta/Orchestration** | workflow-orchestrator, context-manager, claude-code-specialist, sdd-specialist |

All agents live in `claude/agents/`. Orchestration rules defined in `swarm-config.md` and `instructions.md`.

### 3. Skills (8 Domain Skills)

Loaded on-demand from `claude/skills/`:

- `interface-design` — Complex design system work
- `frontend-design` — Simple/one-off page components
- `ux-states` — User experience state modeling
- `security-patterns` — Security implementation patterns
- `compliance-patterns` — Compliance implementation patterns
- `db-migration` — Database migration workflows
- `debugging` — Debugging protocols and investigation
- `sdd-protocols` — SDD process enforcement

### 4. Commands (Slash Commands)

**Custom Clavix Command Overrides** (`claude/commands/clavix/`):

- `product.md` — `/clavix:product` generates `.ops/product-vision-strategy.md`
- `prd.md` — `/clavix:prd` generates `.ops/build/v{x}/prd.md`

**Project-Specific Commands:**

| Group | Commands | Purpose |
|-------|----------|---------|
| Interface Design | init, status, extract, audit | UI design system management |
| Quality Gates | check, report | Quality assurance automation |
| Database | plan | Database migration planning |
| Debugging | investigate | Issue investigation |
| Security | review | Security review |
| Spec Management | validate | Spec completeness validation |
| Vision Management | distill | Split product vision into domain files |
| Orchestration | orchestrate | Main swarm entrypoint |

### 5. MCP Configuration Template

`.mcp.json` — Pre-configured for standard MCPs: GitHub, Supabase, Sentry, Stripe, Context7, Playwright

### 6. .ops/ Directory Structure

All SDD artifacts live in `.ops/`:

```
.ops/
├── product-vision-strategy.md              # Product vision (canonical)
├── quick-product-vision-strategy.md        # Distilled: product context
├── security-compliance-baseline.md         # Distilled: security/compliance
├── tech-architecture-baseline.md           # Distilled: architecture
├── ui-design-system.md                     # UI design system (optional)
└── build/
    ├── system-design.yaml                  # Architecture reference
    └── v{x}/                               # Version-scoped builds
        ├── prd.md                          # Mini-PRD for version
        ├── implementation-status.md        # Progress tracker
        ├── db-migration-plan.yaml          # DB plan (build-level, if needed)
        ├── build-order.yaml                # Cross-feature build order (if needed)
        └── <feature-name>/
            ├── specs.md                    # Feature requirements
            └── tasks.yaml                  # Implementation tasks
```

---

## How Clavix + This Bootstrap Work Together

**Clavix** is used for the **planning phase only** — defining vision, writing PRDs, and improving prompts. **Implementation is handled by `/orchestrate`**, which triggers the agent swarm.

### Clavix Commands Used

| Command | Purpose | Output |
|---------|---------|--------|
| `/clavix:product` | Create product vision | `.ops/product-vision-strategy.md` |
| `/clavix:prd` | Create version-scoped PRD | `.ops/build/v{x}/prd.md` |
| `/clavix:improve` | Analyze and optimize prompts | `.clavix/outputs/prompts/` |

### Implementation: `/orchestrate`

After planning with Clavix, implementation is driven by the `/orchestrate` command.

**Two modes:**

**Planning mode** (creates specs, system-design, tasks):
```
/orchestrate .ops/build/v{x}/
```
Runs Tier 1–2 agents (spec-writer, architect, project-task-planner) for the entire build version.

**Implementation mode** (builds a specific feature):
```
/orchestrate .ops/build/v{x}/<feature-name>/
```
Runs Tier 3–6 agents for a single feature that already has specs.md and tasks.yaml.

---

## SDD Workflow Overview

| Step | Agent/Tool | Output |
|------|-----------|--------|
| 1. Product Vision | You (with Clavix) | `.ops/product-vision-strategy.md` |
| 2. Version PRD | You (with Clavix) | `.ops/build/v{x}/prd.md` |
| 3. Feature Specs | spec-writer | `.ops/build/v{x}/<feature>/specs.md` |
| 4. System Design | architect | `.ops/build/system-design.yaml` |
| 5. DB Migration Plan | database-administrator | `.ops/build/v{x}/db-migration-plan.yaml` (if needed) |
| 6. Implementation Tasks | project-task-planner | `.ops/build/v{x}/<feature>/tasks.yaml` |
| 7. Build Order | workflow-orchestrator | `.ops/build/v{x}/build-order.yaml` (if needed) |
| 8. Implementation | Development agents | Code changes |

---

## MCP Configuration

Add environment variables to `.env` (never commit this file):

```bash
# Supabase
SUPABASE_URL=your-supabase-url
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# GitHub
GITHUB_TOKEN=your-github-token

# Sentry
SENTRY_DSN=your-sentry-dsn

# Stripe
STRIPE_SECRET_KEY=your-stripe-secret-key
```

Verify `.gitignore` excludes `.env`:
```bash
echo ".env" >> .gitignore
```

For Cursor users:
```bash
cp .mcp.json ~/.cursor/mcp.json
```

Validate MCP setup (Claude Code):
```bash
claude mcp list
claude mcp get supabase
```

---

## Adapting to Your Stack

Edit `CLAUDE.md` to reflect your stack:

```markdown
## Stack
Your-Framework, Your-Styling, Your-Backend, Your-Auth
Testing: Your-Test-Framework | Package manager: npm | Workflow: SDD (Clavix + Agent Swarm)
```

Add/remove skills, update agent prompts, and customize MCP servers in `.mcp.json` as needed.

---

## Commands Reference

### Planning Commands (Clavix)

- **`/clavix:product`** — Create Product Vision
- **`/clavix:prd`** — Create Version PRD
- **`/clavix:improve`** — Analyze and Optimize Prompts

### Implementation Command

- **`/orchestrate .ops/build/v{x}/`** — Planning mode (specs, system-design, tasks)
- **`/orchestrate .ops/build/v{x}/<feature-name>/`** — Implementation mode (build specific feature)

### Project-Specific Commands

**Interface Design:**
- `/interface-design:init` — Initialize UI design system
- `/interface-design:status` — Report UI system state
- `/interface-design:extract` — Extract design system from existing UI
- `/interface-design:audit <path>` — Audit UI code against design system

**Quality Gates:**
- `/gate:check` — Run quality checks
- `/gate:report` — Generate quality report

**Database:**
- `/db:plan` — Create database migration plan

**Debugging:**
- `/debug:investigate` — Investigate issues

**Security:**
- `/security:review` — Security review

**Spec Management:**
- `/spec:validate` — Validate spec completeness

**Vision Management:**
- `/vision:distill` — Split product vision into domain-specific files

---

## Support & Contribution

This is a bootstrap template. Adapt it to your needs.

For questions or improvements:
1. Open an issue in this repository
2. Submit a pull request with enhancements
3. Share your customizations with the community

---

## What's Next?

1. Copy the bootstrap to your product repo
2. Run `/clavix:product` to create your product vision
3. Run `/clavix:prd` to define your first version
4. Run `/vision:distill` to split vision into agent-consumable domain files
5. Run `/orchestrate .ops/build/v0/` to kick off planning
6. Run `/orchestrate .ops/build/v0/<feature-name>/` to implement features
7. Iterate and refine your SDD workflow

Happy building with SDD and agent swarms.
