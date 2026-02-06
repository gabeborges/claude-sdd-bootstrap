# claude-sdd-bootstrap

A complete SDD (Specification-Driven Development) toolkit for AI-powered product development with Claude Code and Cursor.

---

## What This Is

This is a **bootstrap/template project**, not a product. It provides a standardized SDD workflow, agent swarm, commands, skills, and guardrails that you copy into your real product repositories.

**Key Pattern: `claude/` → `.claude/`**

- `claude/` — Source templates in this repo (what you copy from)
- `.claude/` — Destination in your product repo (where you copy to)

When adopting SDD in your product, you copy `claude/` contents to `.claude/` in that repo.

---

## Tech Stack

The bootstrap is pre-configured for:

- **Framework**: Next.js (App Router)
- **Styling**: Tailwind, shadcn/ui, Headless UI
- **Backend**: Supabase (auth + database)
- **Payments**: Stripe
- **Auth**: Google OAuth
- **Testing**: Vitest + Playwright
- **Package Manager**: npm
- **Workflow**: SDD (Clavix + Agent Swarm)

You can adapt these defaults in `CLAUDE.md` for your stack.

---

## Prerequisites

Before installing this bootstrap:

1. **Node.js** (v18+) and **npm**
2. **Clavix npm package** — Install globally:
   ```bash
   npm install -g clavix
   ```
3. **Claude Code** (recommended) or **Cursor IDE**
4. **Git** configured with your identity

---

## What's Included

### 1. Guardrails (Runtime Rules)

- **`CLAUDE.md`** — Auto-loaded every session. Defines stack, security rules, coding standards, architecture boundaries, and SDD artifact flow.
- **`AGENTS.md`** — Agent coordination reference. Defines artifact priority table, agent roles, and orchestration rules.

### 2. Agent Swarm (20 Specialized Agents)

All agents live in `claude/agents/`:

**Core Agents:**
- `architect` — System design and architecture decisions
- `spec-writer` — Converts PRD features into detailed specs
- `project-task-planner` — Breaks specs into implementation tasks
- `workflow-orchestrator` — Coordinates multi-agent workflows
- `context-manager` — Manages context window and token budget

**Development Agents:**
- `fullstack-developer` — Full-stack implementation
- `frontend-designer` — Frontend component design
- `ui-designer` — UI/UX design and interface work
- `database-administrator` — Database schema and migrations

**Quality Agents:**
- `code-reviewer` — Code quality and standards enforcement
- `qa` — Quality assurance and testing strategy
- `test-automator` — Automated test creation
- `debugger` — Issue investigation and debugging

**Security & Compliance:**
- `security-engineer` — Security implementation
- `security-auditor` — Security audit and review
- `compliance-engineer` — Compliance implementation
- `compliance-auditor` — Compliance audit and review

**Meta Agents:**
- `claude-code-specialist` — Claude Code configuration and optimization
- `sdd-specialist` — SDD process and documentation

**Orchestration Files:**
- `swarm-config.md` — Agent tier definitions and coordination rules
- `instructions.md` — Swarm behavior and execution guidelines

### 3. Skills (8 Domain Skills)

Skills live in `claude/skills/` and are loaded on-demand:

- `interface-design` — Complex design system work
- `frontend-design` — Simple/one-off page components
- `ux-states` — User experience state modeling
- `security-patterns` — Security implementation patterns
- `compliance-patterns` — Compliance implementation patterns
- `db-migration` — Database migration workflows
- `debugging` — Debugging protocols and investigation
- `sdd-protocols` — SDD process enforcement

### 4. Commands (Slash Commands)

Commands live in `claude/commands/` and provide workflow automation.

**Custom Clavix Command Overrides** (`claude/commands/clavix/`):

These override default Clavix npm package behavior to enforce SDD-specific workflows:

- `product.md` — `/clavix:product` generates `.ops/product-vision-strategy.md`
  - High-level product vision and technical strategy
  - Long-term direction, principles, constraints
  - Referenced by all PRDs and architectural decisions

- `prd.md` — `/clavix:prd` generates `.ops/build/v{x}/prd.md`
  - Version-scoped Mini-PRD
  - Checks for existing product vision
  - Focused questions about users, problems, features, metrics
  - Scope limited to one version/epic

**Project-Specific Commands:**

- **Interface Design** (`interface-design/`)
  - `init.md` — Initialize UI design system
  - `status.md` — Report current UI system state
  - `extract.md` — Extract design system from existing UI
  - `audit.md` — Audit UI code against design system

- **Quality Gates** (`gate/`)
  - `check.md` — Run quality checks
  - `report.md` — Generate quality report

- **Database** (`db/`)
  - `plan.md` — Create database migration plan

- **Debugging** (`debug/`)
  - `investigate.md` — Investigate issues

- **Security** (`security/`)
  - `review.md` — Security review

- **Spec Management** (`spec/`)
  - `validate.md` — Validate spec completeness

- **Vision Management** (`vision/`)
  - `distill.md` — Split product vision into domain-specific files

- **Orchestration** (`orchestrate.md`)
  - `/orchestrate .ops/build/v{x}/<feature-name>/` — Main swarm entrypoint for implementation

### 5. MCP Configuration Template

`.mcp.json` — Pre-configured for standard MCPs:

- GitHub
- Supabase
- Sentry
- Stripe
- Context7
- Playwright

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

This bootstrap uses three Clavix commands:

- `/clavix:product` — Create product vision (custom override in `claude/commands/clavix/product.md`)
- `/clavix:prd` — Create version-scoped PRD (custom override in `claude/commands/clavix/prd.md`)
- `/clavix:improve` — Analyze and optimize prompts (built-in from Clavix npm package)

### Implementation: `/orchestrate`

After planning with Clavix, implementation is driven by the `/orchestrate` command:

```
/orchestrate .ops/build/v{x}/<feature-name>/
```

This is the **main entrypoint** for the agent swarm. It reads the feature specs and tasks, assigns agents by tier, and executes the full build pipeline. Use it right after `/clavix:prd` to kick off the SDD artifact chain and implementation.

### Why Custom Overrides?

The default Clavix commands are general-purpose. The custom `product.md` and `prd.md` overrides enforce:

- Specific output paths (`.ops/` directory structure)
- Product vision before PRD workflow
- Version scoping for PRDs
- SDD artifact chain enforcement
- Integration with agent swarm

---

## SDD Workflow Overview

The complete SDD artifact chain:

1. **Product Vision** (`/clavix:product`)
   - Output: `.ops/product-vision-strategy.md`
   - Who: You (with Clavix guidance)
   - What: High-level vision, principles, technical strategy

2. **Version-Scoped PRD** (`/clavix:prd`)
   - Output: `.ops/build/v{x}/prd.md`
   - Who: You (with Clavix guidance)
   - What: Version scope, features, success metrics

3. **Feature Specs** (spec-writer agent)
   - Output: `.ops/build/v{x}/<feature>/specs.md`
   - Who: `spec-writer` agent
   - What: Detailed requirements, acceptance criteria

4. **System Design** (architect agent)
   - Output: `.ops/build/system-design.yaml`
   - Who: `architect` agent
   - What: Architecture decisions, component design

5. **Database Migration Plan** (database-administrator agent, conditional)
   - Output: `.ops/build/v{x}/db-migration-plan.yaml` (one consolidated plan per build version)
   - Who: `database-administrator` agent
   - What: Schema changes, migration steps (only for DB-touching features)

6. **Implementation Tasks** (project-task-planner agent)
   - Output: `.ops/build/v{x}/<feature>/tasks.yaml`
   - Who: `project-task-planner` agent
   - What: Actionable tickets with spec pointers

7. **Build Order** (workflow-orchestrator, conditional)
   - Output: `.ops/build/v{x}/build-order.yaml` (cross-feature execution order)
   - Who: `workflow-orchestrator` agent
   - What: Determines feature build sequence when there are cross-feature dependencies

8. **Implementation** (`/orchestrate .ops/build/v{x}/<feature-name>/`)
   - Output: Code changes
   - Who: Development agents (coordinated by workflow-orchestrator)
   - What: The orchestrator loads specs, assigns agents by tier, runs quality gates, and produces working software

---

## Quick Start

Get started in 5 minutes:

1. **Install Clavix**:
   ```bash
   npm install -g clavix
   ```

2. **Clone this bootstrap**:
   ```bash
   git clone https://github.com/your-username/claude-sdd-bootstrap.git
   cd claude-sdd-bootstrap
   ```

3. **Copy to your product repo**:
   ```bash
   # In your product repo root:
   cp -r /path/to/claude-sdd-bootstrap/claude/ ./.claude/
   cp /path/to/claude-sdd-bootstrap/CLAUDE.md ./
   cp /path/to/claude-sdd-bootstrap/AGENTS.md ./
   cp /path/to/claude-sdd-bootstrap/.mcp.json ./
   ```

4. **Configure MCP** (add env vars to `.env`, never commit):
   ```bash
   # Add to .env:
   SUPABASE_URL=...
   SUPABASE_SERVICE_ROLE_KEY=...
   GITHUB_TOKEN=...
   # etc.
   ```

5. **Start SDD workflow**:
   ```bash
   # In Claude Code or Cursor:
   /clavix:product                              # Create product vision
   /clavix:prd                                  # Create version PRD
   /orchestrate .ops/build/v0/<feature-name>/   # Agent swarm builds it
   ```

---

## Installation Guide (Detailed)

### Step 1: Prerequisites

Ensure you have:
- Node.js v18+ and npm installed
- Clavix installed globally: `npm install -g clavix`
- Claude Code or Cursor IDE
- Git configured with your identity

### Step 2: Clone Bootstrap

```bash
git clone https://github.com/your-username/claude-sdd-bootstrap.git
cd claude-sdd-bootstrap
```

### Step 3: Copy to Product Repo

In your product repository root:

```bash
# Copy agent swarm, skills, and commands
cp -r /path/to/claude-sdd-bootstrap/claude/ ./.claude/

# Copy guardrails
cp /path/to/claude-sdd-bootstrap/CLAUDE.md ./
cp /path/to/claude-sdd-bootstrap/AGENTS.md ./

# Copy MCP configuration
cp /path/to/claude-sdd-bootstrap/.mcp.json ./

# Create .ops directory structure
mkdir -p .ops/build
```

### Step 4: Configure MCP Servers

1. Add environment variables to `.env` (never commit this file):
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

   # Add other MCP-required env vars
   ```

2. Verify `.gitignore` excludes `.env`:
   ```bash
   echo ".env" >> .gitignore
   ```

3. For Cursor users, also copy to:
   ```bash
   cp .mcp.json ~/.cursor/mcp.json
   ```

4. Validate MCP setup (Claude Code):
   ```bash
   claude mcp list
   claude mcp get supabase
   ```

### Step 5: Customize for Your Stack

Edit `CLAUDE.md` if your stack differs:

```markdown
## Stack
Your-Framework, Your-Styling, Your-Backend, Your-Auth
Testing: Your-Test-Framework | Package manager: npm | Workflow: SDD (Clavix + Agent Swarm)
```

### Step 6: Start Workflow

In Claude Code or Cursor:

1. Create product vision:
   ```
   /clavix:product
   ```

2. Create version PRD:
   ```
   /clavix:prd
   ```

3. Orchestrate implementation (planning + build via agent swarm):
   ```
   /orchestrate .ops/build/v0/<feature-name>/
   ```

---

## Directory Structure

### Source Templates (This Repo)

```
claude-sdd-bootstrap/
├── CLAUDE.md                          # Guardrails (copy to product repo root)
├── AGENTS.md                          # Agent coordination (copy to product repo root)
├── .mcp.json                          # MCP config template (copy to product repo root)
├── claude/                            # Source templates (copy to .claude/ in product repo)
│   ├── agents/                        # 20 specialized agents
│   │   ├── architect.md
│   │   ├── spec-writer.md
│   │   ├── project-task-planner.md
│   │   ├── workflow-orchestrator.md
│   │   ├── context-manager.md
│   │   ├── fullstack-developer.md
│   │   ├── frontend-designer.md
│   │   ├── ui-designer.md
│   │   ├── database-administrator.md
│   │   ├── code-reviewer.md
│   │   ├── qa.md
│   │   ├── test-automator.md
│   │   ├── debugger.md
│   │   ├── security-engineer.md
│   │   ├── security-auditor.md
│   │   ├── compliance-engineer.md
│   │   ├── compliance-auditor.md
│   │   ├── claude-code-specialist.md
│   │   ├── sdd-specialist.md
│   │   ├── swarm-config.md
│   │   └── instructions.md
│   ├── skills/                        # 8 domain skills
│   │   ├── interface-design/
│   │   ├── frontend-design/
│   │   ├── ux-states/
│   │   ├── security-patterns/
│   │   ├── compliance-patterns/
│   │   ├── db-migration/
│   │   ├── debugging/
│   │   └── sdd-protocols/
│   └── commands/                      # Slash commands
│       ├── clavix/                    # Custom Clavix overrides
│       │   ├── product.md             # Override: /clavix:product
│       │   └── prd.md                 # Override: /clavix:prd
│       ├── interface-design/          # UI design system commands
│       ├── gate/                      # Quality gate commands
│       ├── db/                        # Database commands
│       ├── debug/                     # Debugging commands
│       ├── security/                  # Security commands
│       ├── spec/                      # Spec management
│       ├── vision/                    # Vision management
│       └── orchestrate.md             # Swarm entrypoint
```

### Product Repo (After Installation)

```
your-product-repo/
├── CLAUDE.md                          # Guardrails
├── AGENTS.md                          # Agent coordination
├── .mcp.json                          # MCP configuration
├── .claude/                           # Copied from bootstrap claude/
│   ├── agents/
│   ├── skills/
│   └── commands/
├── .ops/                              # SDD artifacts (created during workflow)
│   ├── product-vision-strategy.md
│   ├── quick-product-vision-strategy.md
│   ├── security-compliance-baseline.md
│   ├── tech-architecture-baseline.md
│   ├── ui-design-system.md
│   └── build/
│       ├── system-design.yaml
│       └── v0/
│           ├── prd.md
│           ├── implementation-status.md
│           ├── db-migration-plan.yaml
│           ├── build-order.yaml
│           └── feature-name/
│               ├── specs.md
│               └── tasks.yaml
└── app/                               # Your product code
```

---

## Commands Reference

### Planning Commands (Clavix)

**`/clavix:product`** — Create Product Vision (custom override)
- Output: `.ops/product-vision-strategy.md`
- Purpose: High-level vision, principles, technical strategy
- Run once per product (or when vision changes)

**`/clavix:prd`** — Create Version PRD (custom override)
- Output: `.ops/build/v{x}/prd.md`
- Purpose: Version-scoped features and success metrics
- Run once per version/epic

**`/clavix:improve`** — Analyze and Optimize Prompts (built-in)
- Purpose: Quality assessment and prompt optimization
- Saves improved prompts to `.clavix/outputs/prompts/`

### Implementation Command

**`/orchestrate .ops/build/v{x}/<feature-name>/`** — Run Agent Swarm
- Purpose: Main implementation entrypoint — triggers the full SDD pipeline
- Reads feature specs and tasks, assigns agents by tier, executes build
- Use right after `/clavix:prd` to begin planning and implementation

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

## Agent Swarm Overview

### Agent Tiers

Agents operate in tiers to ensure proper dependencies:

**Tier 1: Planning & Architecture**
- `architect` — System design
- `spec-writer` — Feature specifications
- `project-task-planner` — Task breakdown

**Tier 2: Implementation**
- `fullstack-developer` — Full-stack development
- `frontend-designer` — Frontend components
- `ui-designer` — UI/UX design
- `database-administrator` — Database work

**Tier 3: Quality & Security**
- `code-reviewer` — Code review
- `qa` — Quality assurance
- `test-automator` — Test creation
- `security-engineer` — Security implementation
- `compliance-engineer` — Compliance implementation

**Tier 4: Audit & Validation**
- `security-auditor` — Security audit
- `compliance-auditor` — Compliance audit
- `debugger` — Issue investigation

**Meta Tier: Orchestration & Support**
- `workflow-orchestrator` — Multi-agent coordination
- `context-manager` — Context window management
- `claude-code-specialist` — Claude Code optimization
- `sdd-specialist` — SDD process support

### Agent Coordination

Agents follow strict read/write contracts defined in `AGENTS.md`:

1. **Read artifacts before coding** — Check specs, system design, PRD
2. **Escalate ambiguity** — Never guess, ask questions
3. **Write to designated outputs** — Respect artifact ownership
4. **No cross-tier dependencies** — Lower tiers cannot block higher tiers

### Running the Swarm

```bash
# In Claude Code or Cursor:
/orchestrate .ops/build/v0/<feature-name>/
```

The orchestrator will:
1. Load feature specs
2. Assign agents to tiers
3. Execute tier-by-tier
4. Run quality gates
5. Create implementation artifacts

---

## Customization

### Adapting to Your Stack

1. Edit `CLAUDE.md` to reflect your stack
2. Update agent prompts if needed
3. Add/remove skills based on your domain
4. Customize MCP servers in `.mcp.json`

### Adding Custom Agents

1. Create agent file in `claude/agents/<agent-name>.md`
2. Define YAML frontmatter (name, description, tools, model)
3. Write focused system prompt
4. Update `claude/agents/swarm-config.md` with tier assignment
5. Reference in `AGENTS.md` if needed

### Adding Custom Skills

1. Create skill directory: `claude/skills/<skill-name>/`
2. Add `SKILL.md` with YAML frontmatter + instructions
3. Keep description short (always in context)
4. Put detail in body (loaded on-demand)

### Adding Custom Commands

1. Create command file: `claude/commands/<group>/<name>.md`
2. Define YAML frontmatter (name, description, model, tools)
3. Write instructions with `$ARGUMENTS` for dynamic input
4. Test in `.claude/` before distributing

---

## Support & Contribution

This is a bootstrap template. Adapt it to your needs.

For questions or improvements:
1. Open an issue in this repository
2. Submit a pull request with enhancements
3. Share your customizations with the community

---

## What's Next?

1. Install the bootstrap in your product repo
2. Run `/clavix:product` to create your product vision
3. Run `/clavix:prd` to define your first version
4. Run `/orchestrate .ops/build/v0/<feature-name>/` to kick off the agent swarm
5. Iterate and refine your SDD workflow

Happy building with SDD and agent swarms.
