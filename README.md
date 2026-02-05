# This file version is deprecated. Changes in progress...


# SDD + Agent Swarm Bootstrap

This repository is **not a product**. It is a **bootstrap project** that defines **process, agents, commands, and guardrails** to be installed into real product repositories.

It standardizes:
- Spec-Driven Development (SDD)
- Clavix → OpenSpec → Agent Swarm flow
- Claude Code + Cursor usage
- MCP server integration
- Non-negotiable engineering and security rules

You copy/install parts of this repo into each product repo.

---

## What You Install Into Product Repos

### 1. Claude Commands
**Pre-prequisite: install [Clavix](https://github.com/ClavixDev/Clavix/tree/main) and [OpenSpec](https://github.com/Fission-AI/OpenSpec).**

Copy into the product repo:
```
.claude/commands/
├── clavix/             # product.md, prd.md
├── openspec/           # proposal.md
├── interface-design/   # init.md, status.md, extract.md, audit.md
└── orchestrate         # swarm entrypoint
```

Commands:
- `/clavix:product` → `.ops/build/product-vision-strategy.md`
- `/clavix:prd` → `.ops/build/v{x}/prd.md`
- `/openspec:proposal` → `.ops/build/v{x}/epic.md`
- `/orchestrate` → agent swarm implementation
- `/openspec:apply` → deprecated

- `/interface-design:init` → initialize `.ops/ui-design-system.md`
- `/interface-design:status` → report current UI design system state
- `/interface-design:extract` → extract/update `.ops/ui-design-system.md` from existing UI
- `/interface-design:audit <path>` → audit UI code against `.ops/ui-design-system.md`

### 1.1 Claude Skills
Copy into the product repo:
```
.claude/skills/
├── interface-design/    # SKILL.md + references/ (for complex design systems and pages)
└── frontend-design/     # SKILL.md (for one-off / simple pages)
```

Notes:
- If `.ops/ui-design-system.md` exists, UI changes MUST comply with it.
- For simple one-off pages/components (no system work), use the `frontend-design` skill.

---

### 2. Global Guardrails (Required)
Copy to product repo root:
```
AGENTS.md   # source of truth (rules + swarm collaboration)
CLAUDE.md   # runtime execution guardrails
```

If files conflict, **AGENTS.md wins**.

---

### 3. Agent Swarm (Required)
Copy to .claude folder:
```
.claude/agents/
```

Must include (do not rename):
- `.claude/agents/instructions.md`
- `.claude/agents/swarm-config.md`

Defines agent roles, tiers, and orchestration rules.

---

### 4. OpenSpec Integration
For OpenSpec-enabled repos:
```
openspec/AGENTS.md
```
(renamed from `AGENTS.openspec.md`)

Ensures OpenSpec outputs match `.ops/build/...` and hand off cleanly to the swarm.

---

### 5. MCP Configuration
Standard MCPs:
- GitHub, Supabase, Sentry, Stripe, Context7, Playwright

Install in product repo:
```
.mcp.json
```

Also copy to:
```
~/.cursor/mcp.json
```

Add env vars to `.env` (never commit).

Workaround for known Claude Code UI bug:
```bash
claude mcp add playwright --transport stdio --scope local -- npx -y @playwright/mcp@latest
```

Then validate:
```bash
claude mcp list
claude mcp get supabase
```

---

## SDD Workflow (in Product Repos)

1. `/clavix:product` **[NEW]** - Generates high level product vision and tech definitions `product-vision-strategy.md`
2. `/clavix:prd` **[UPDATED]** - Generates build specific `prd.md` document
3. `/openspec:proposal` **[UPDATED]** - Generates build specific `epic.md` and feature level `spec.md` documents
4. `/interface-design:init` **[OPTIONAL]** - Generates ui system design for the project (use frontend-design skill for simple pages)
5. `/orchestrate .ops/build/v{x}/<feature-name>/` **[NEW]** - Generates feature level `tasks.md` and executes on plan

The orchestrator enforces:
- spec → tasks → code
- MCP preflight + snapshots
- tiered agent execution
- no guessing or drift
