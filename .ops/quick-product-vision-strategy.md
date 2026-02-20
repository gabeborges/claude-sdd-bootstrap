<!-- Source: .ops/product-vision-strategy.md -->
<!-- Included: §1, §2, §3, §4, §5, §6, §7, §12 -->
<!-- Skipped: none -->
<!-- Regenerate: /vision:distill -->

## 1. Vision

Enable small therapy practices to run their entire business from Google Workspace, eliminating the need for separate practice management tools.

**Long-term platform vision:** Enable regulated teams (starting with therapy practices) to use AI and modern tools within Google Workspace without risking compliance, trust, or future rework.

**Product evolution strategy:**
- **Practice Management (Wedge)** → Solve immediate, daily problems (notes, scheduling, billing, communication); prove trust with sensitive data; establish Curio as part of core operations
- **Policy Management (Control Layer)** → Encode compliance rules as system policies; govern access, data flow, retention, and AI behavior
- **Compliance Infrastructure (Platform)** → Abstract policies into reusable primitives; provide auditability, enforcement, and verification at scale; become the trusted compliance layer for AI-enabled work across regulated industries

---

## 2. Problem Space

**Primary pain points:**
1. Solo online therapists spend too much time on admin tasks (therapy notes, processing payments, sending invoices, insurance billing, communicating with clients, managing their agenda)
2. Existing practice management tools are expensive for solo practitioners
3. Existing tools are complex and too generic, not covering online therapy use cases (group therapy, insurance billing, workflows)
4. Therapists are familiar with Google Workspace and spend most of their time in it (Gmail, Calendar, Google Docs)

**Who experiences these problems:** Solo online therapists (one therapist, fully remote practice) and solo therapists (one therapist, hybrid practice)

**Why existing solutions fall short:**
- Too expensive for solo practitioners
- Too complex/feature-heavy
- Too generic, not built for online therapy workflows
- Require switching away from tools therapists already use (Google Workspace)

---

## 3. Target Customers

**Primary target:** Solo online therapists (one therapist, fully remote practice)

**Secondary target:** Solo therapists (one therapist, hybrid practice)

**Focus:** Primary and secondary for now

**Future target:** Small practices (2-5 therapists, hybrid practice) - later

**Key characteristics:**
- Familiar with Google Workspace (Gmail, Calendar, Google Docs)
- Spend most of their time in Google Workspace
- Need online therapy-specific features (group therapy, insurance billing, workflows)
- Cost-sensitive (solo practitioners)

---

## 4. Value Proposition

**Core Benefits:**
- **Everything in one place** - Sessions, notes, calendar, documents, and client communication live together — no tool juggling, no copy-pasting, no missed details
- **Compliance without the stress** - Curio is designed to meet healthcare privacy requirements by default, so you don't have to figure out settings, policies, or legal details on your own
- **AI that actually saves time** - Notes, summaries, and admin support are automated in a way that's safe for client data — helping you reclaim hours each week
- **Works with tools you already use** - Curio fits naturally into Google Workspace, so you don't need to learn an entirely new system or migrate everything
- **Built for online therapy** - Designed from day one for telehealth, not retrofitted from in-person workflows

**Why Not the Alternatives?**

Most alternatives either:
- Focus on features but leave compliance up to you
- Add AI without clear data protections
- Require multiple disconnected tools
- Feel heavy, outdated, or built for clinics—not solo therapists

**Differentiation:** Curio is different because it's simple on the surface and safe underneath.

**One-Line Value Proposition:** Curio helps solo online therapists run their practice and save time — without worrying about compliance, tech complexity, or client data safety.

**Platform vision (long-term):**
- **Compliance by default** - Safety and regulatory guarantees are built into the system, not left to settings or training
- **AI you can actually trust** - AI operates within clear, auditable boundaries, making it safe for real, regulated work
- **No new silos** - Curio governs work inside tools you already use (like Google Workspace), reducing risk and complexity
- **Built to scale safely** - Growth doesn't increase compliance burden; audits and governance get easier over time

**Intentional Trade-Offs:**
- Less flexibility, more certainty - You can't override safety-critical rules
- Fewer flashy features - Only AI and automations that are safe in production
- Opinionated by design - Strong defaults so you don't need to be a compliance expert

---

## 5. Strategic Pillars

1. **Fits Into Therapists' Daily Work** - Curio adapts to existing workflows and tools (Google Workspace and others) instead of forcing therapists into a new silo

2. **Compliance by Default** - Regulatory and privacy requirements are enforced by the system, not left to configuration or user behavior

3. **AI with Clear Boundaries** - AI is policy-bound, auditable, and safe for real clinical use

4. **Radical Simplicity for Solo Therapists** - Every feature must reduce admin time and cognitive load

5. **Built to Scale Beyond Practice Management** - Product decisions must support Curio's evolution into a broader compliance platform

---

## 6. Success Metrics

**The Single Proof It's Working:** Therapists stop thinking about compliance and admin — and simply run their practice through Curio every week.

**North Star Metric:** Weekly Active Therapists running core workflows end-to-end in Curio (e.g. sessions → notes → follow-ups → billing, without leaving their existing tools)

If Curio is working, therapists default to it to run their practice.

**Leading Indicators (Wedge: Therapy Practices):**
- Net admin time saved per therapist per week → Direct signal of real value delivered
- 30/90-day retention of solo therapists → Indicates Curio is embedded in daily work
- AI feature adoption within compliant workflows → Signals trust, not just curiosity
- Self-reported compliance confidence ("I feel safe using Curio with client data") → Measures peace of mind, not feature usage
- Revenue per active therapist → Confirms willingness to pay for value, not just access

**Long-Term Platform Signals:**
- % of workflows governed by explicit policies → Measures transition from features to compliance primitives
- Number of reusable policy types supported → Indicates platform maturity
- Expansion into adjacent regulated roles (without rebuilding core systems) → Proof of horizontal scalability

---

## 7. Non-Goals

- Not for large clinics/hospitals
- Not for non-regulated industries
- Not a full EHR
- Not a billing-only tool
- Not a general productivity tool
- Not supporting non-Google Workspace platforms (initially)

---

## 12. AI Strategy

**Role of AI:** AI is an assistive copilot, not an autonomous clinician. It accelerates admin work (notes, summaries, follow-ups, billing drafts) while staying policy-bound and auditable.

**Non-Negotiable Boundaries & Guarantees:**

- **Policy-bound execution:** AI can only read/write what the policy engine allows for that actor/context
- **Explainability + provenance:** Every output links to inputs/sources + the policy decision that permitted it
- **No silent actions:** AI never sends, files, bills, shares, or deletes without explicit user confirmation (or an admin-approved, policy-defined automation)
- **No training on customer PHI:** Customer data is not used to train shared models
- **Fail-closed:** If policy, identity, or context is unclear, AI refuses or asks

**Must Always Have Human Oversight:**

- Clinical notes finalization (sign-off required)
- Any external communication (emails/messages to clients)
- Billing/claims submission and financial actions
- Sharing/exporting PHI outside Curio (Drive, email, third parties)
- Permission/policy changes and retention/deletion actions

**What AI Explicitly Cannot Do:**

- Diagnose, prescribe, or provide clinical advice as a professional replacement
- Change policies/permissions, bypass access controls, or expand its own scope
- Access data across clients/tenants beyond authorized context
- Create "new workflows" that aren't defined in product + policy

**Fit with Compliance-First Architecture:**

- AI is treated as a constrained service behind the policy engine
- All AI operations emit audit events (inputs, outputs, model/provider, purpose, actor, policy decision)
- Outputs are draft artifacts until user approval (human-in-the-loop as a default safety rail)

**Evolution Across the 3 Stages:**

- **Practice management:** Assistive drafting + summaries + task suggestions; heavy human review
- **Policy management:** AI becomes "policy-aware" (suggests next steps that comply; flags risky actions)
- **Compliance platform:** AI helps generate evidence, audits, and policy checks (e.g., "show all exports last 30 days"), still policy-bound and non-autonomous for high-risk actions
