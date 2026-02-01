# README

## Commands customization
These files need to be pasted inside `.claude/commands/<tool-name>` (eg: `.claude/commands/clavix`)
- `/clavix:product` [NEW] (Creates high level `product-vision-strategy.md` document) 
- `/clavix:prd` [UPDATE] (Creates a build version `prd.md` document)
- `/openspec:proposal` [UPDATE] (Better handoff with Clavix)
- `/openspec:apply` [UPDATE] (Better handoff with Clavix. Replaced by agents swarm)
- `/orchestrate` [NEW] (Agent swarm orchestration for implementation)

## Clavix customization
1. Add files `prd.md` and `product.md` to `.claude/commands/clavix`.
2. Type command `/clavix:product` for high level product vision and strategy definition (includes tech constraints and other details) (future plan to break it down into different artifacts `security-baseline.md`, `compliance-baseline.md`, `tech-stack-constraints.md`, `system-invariants.md`).
3. Type command `/clavix:prd` to create a build/project lvl `prd.md` file that will be consumed by agents.

## OpenSpec customization
1. Add file `proposal.md` to `.claude/commands/openspec`.
2. Type command `/openspec:proposal` in Claude terminal.
3. Paste `AGENTS.openspec.md` to `openspec` folder and rename it to `AGENTS.md`. This file has a proper handoff with OpenSpec and agents swarm and it's following the SDD conventions for this project.

## CLAUDE.md
1. Paste `CLAUDE.md` to project's root folder. This file contains global security, identity, & baseline defaults.
2. Paste `CLAUDE.project.md` to X folder and rename it to `CLAUDE.md`. This file contains execution guardrails (SDD + Next.js + stack).

## Agents configuration
1. Paste `AGENTS.md` to project's root folder. This file contains the agents' global rules + swarm collaboration.
2. Paste folder `agents` to the project's root folder.
- `agents/instructions.md` contains general guidance for agents. It's reffered by agents and other files. Do NOT rename it.
- `agents/swarm-config.md` contains configurations on agent and teammate mapping to orientate swarm of agents.  It's reffered by agents and other files. Do NOT rename it.

## MCP servers configuration
1. Add `mcp.json` on project root level (for Claude).
2. Add the same `mcp.json` to `~/.cursor/mcp.json` Cursor in Cursor Settings > Tools & MCP.
3. Add the same `mcp.json` to `.cursor/mcp.json` project folder.
4. Add environment variables to `.env` file.
5. Add one server to local scope eg: `claude mcp add playwright --transport stdio --scope local -- npx -y @playwright/mcp@latest`.
6. Restart Claude interactive interface (the Claude terminal inside Cursor).
7. On Claude interactive interface type `/mcp` or `claude mcp list`.

