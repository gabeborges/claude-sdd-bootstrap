## TO-DO
[?] test manually [.ops/analysis/agent-governance-report.md](.ops/analysis/agent-governance-report.md)[.clavix/outputs/prompts/std-20260205-120000-g7x2.md]
[x] test agent swarm
[x] get files created in the new setup (setup-6) and ask AI to analyse and tell why and for what purpose AI generated these files. The idea is to understand the real need for AI creating these files.
[?] v0 setup (tools, environments including local (how to run locally), seed generation) [.clavix/outputs/prompts/std-20260206-120000-v0bs.md](.clavix/outputs/prompts/std-20260206-120000-v0bs.md) [.ops/analysis/v0-bootstrap-implementation-plan.md](.ops/analysis/v0-bootstrap-implementation-plan.md)
[-] generate CLAUDE.md based on product-vision-strategy and agent-swarm.md and instructions.md and orchestration.md
[x] Makes README.md shorter and more actionable with less explanations
[x] README.md: fix Orchestrate implementation (planning + build via agent swarm): /orchestrate .ops/build/v0/<feature-name>/ because for planning it should start with v0 and then implemetnation will be <feature-name>
[ ] whenever a file is created by an agent, it should ALWAYS contain their name, data of creation and update date. ALWAYS
[?] clavix to always give pros and cons when presenting approaches to choose from (even here)
[ ] does security.yaml and compliance.yaml means a chagne in upstream files is needed?
[ ] bug fixes needs to update upstream files like database-migration.yaml or specs.md or tasks.md 
[ ] checks.yaml is being created inside features when it should be at the build lvl
[x] implement token efficiency changes [.ops/analysis/token-efficiency-report.md](.ops/analysis/token-efficiency-report.md)
[ ] make the fullstack dev into a nextjs dev? 
[ ] Add nextjs/react code improvements

## DONE:
[x] Add architect agent to the workflow as a planning agent (see full list of agents in /agents)
[x] system-design.md and tasks.md should be an .yaml file so it's more performatic for AI (keep spec.md as md)
[x] If there are no system-design.yaml file, it should be created by the architect before tasks being created (fix the workflow)
[x] The flow is specs.md -> system design.yaml -> tasks.yaml
[x] PRD should live in `.ops/build/v{x}/prd.md`
[x] proposal.md, acceptance.md, epic.md were removed
[x] check spec_change_requests file mention and fix it
[x] breakdown product-vision-strategy document (tech stack, compliance-baseline, security-baseline, etc) <- need to map them
[?] create a product-manager agent to handle product-vision-strategy and other product definitions?
[-] (dropped) create an agent to breakdown product-vision-strategy or breakdown the /clavix:product command?
[x] ask ai to analyse product-vision and prd to see if there are any information that is relevant to agents there and define who should read and extract it
[x] extract skills and commands from agents
[x] Improve README.md file
[x] specs.md cannot be created if prd.md does not exist
[x] system-design.yaml cannot be created if specs.md do not exist
[x] taks.yaml cannot be created if system-design.yaml and specs.md does not exist
[x] (dependency) break down product-vision-strategy into smaller focused files (what happens if during /clavix:product these sections are not filled?) [.ops/analysis/vision-document-efficiency-report.md](.ops/analysis/vision-document-efficiency-report.md)[.clavix/outputs/prompts/comp-20260206-120000-v8d3.md](.clavix/outputs/prompts/comp-20260206-120000-v8d3.md) implementation: [.clavix/outputs/prompts/comp-20260206-120000-vd01.md](.clavix/outputs/prompts/comp-20260206-120000-vd01.md)
[x] product-vision-strategy and prd should be consumed by some agents (has testing strategy) [.ops/analysis/agent-governance-report.md](.ops/analysis/agent-governance-report.md)[.clavix/outputs/prompts/std-20260205-120000-g7x2.md](.clavix/outputs/prompts/std-20260205-120000-g7x2.md) .clavix/outputs/prompts/comp-20260206-163000-r8k4.md
[x] (dependency) creates SDD specialist agent. Needs to be evoked (don't want it to run all the time) [.clavix/outputs/prompts/std-20260205-160500-s2d7.md](.clavix/outputs/prompts/std-20260205-160500-s2d7.md)
[x] DB migration in sdd process[.clavix/outputs/prompts/comp-20260205-153000-m4k8.md](.clavix/outputs/prompts/comp-20260205-153000-m4k8.md)
[x] DB migration plan should be a build level or task level? 
[x] SDD agent to review all files to check if SDD is being followed and if it was implemented well (any gaps od flaws)

## IDEAS (to groom)
[ ] improve orchestration command:
- :plan (run planning session)
- :build (run building session)
- :distill 
- :agent-name (call specific agent for a single task)
- :chat to discuss ideas or implementation paths
- :continue to continue working on the previous implementation that was paused for some reason
- :analyze, :review
[ ] if there is a change in product-vision-strategy document then it should trigger downstream updates
