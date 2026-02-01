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
├── clavix/          # product.md, prd.md
├── openspec/        # proposal.md
└── orchestrate.md   # swarm entrypoint
```

Commands:
- `/clavix:product` → `.ops/build/product-vision-strategy.md`
- `/clavix:prd` → `.ops/build/v{x}/prd.md`
- `/openspec:proposal` → `.ops/build/v{x}/epic.md`
- `/orchestrate` → agent swarm implementation
- `/openspec:apply` → deprecated

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
Copy to product repo root:
```
agents/
```

Must include (do not rename):
- `agents/instructions.md`
- `agents/swarm-config.md`

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

1. `/clavix:product`
2. `/clavix:prd`
3. `/openspec:proposal`
4. `/orchestrate .ops/build/v{x}/<feature-name>/`

The orchestrator enforces:
- spec → tasks → code
- MCP preflight + snapshots
- tiered agent execution
- no guessing or drift
