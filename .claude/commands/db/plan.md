---
name: db:plan
description: Plan a database migration with expand/contract pattern and risk assessment
---

# DB Migration Plan

Plan a safe database migration using the expand/contract pattern.

## Required Reading

Before planning, load `.claude/skills/db-migration/SKILL.md` for patterns, risk assessment, and the output schema.

## Usage

```
/db:plan $ARGUMENTS
```

`$ARGUMENTS` should describe the schema change needed. Examples:
- "Add email_verified boolean to users table"
- "Rename user_name to display_name in profiles"
- "Split address fields from users into addresses table"

If omitted, prompt the user to describe the schema change.

## Process

### Step 1: Understand Current Schema

Read relevant migration files or schema to understand the current state. If you cannot determine the current schema, ask the user.

### Step 2: Plan Migration Phases

For the requested change, produce a plan with:
1. **Expand** phase: additive changes only
2. **Migrate** phase: data backfill if needed
3. **Contract** phase: destructive changes (flag as requiring approval)

Reference `.claude/skills/db-migration/references/patterns.md` for examples.

### Step 3: Assess Risk

Use the risk assessment matrix from the skill:
- Data volume impact
- Downtime risk
- Rollback complexity
- Application impact

### Step 4: Define Rollback

For each migration phase, document the rollback procedure.

### Step 5: Output

Write the migration plan to the feature workspace:

Path: `.ops/build/v{x}/<feature-name>/db-migration-plan.yaml`

Use the schema from the db-migration skill. If no feature workspace exists, output the plan inline.

```yaml
db_change:
  required: true
  migrations:
    - phase: expand
      sql: "..."
    - phase: migrate
      sql: "..."
    - phase: contract
      sql: "..."
      requires_approval: true
  data_backfill:
    required: true|false
    steps: []
  risk:
    level: low|med|high
    notes: []
  rollback:
    steps: []
```

### Step 6: Supabase Reminders

After outputting the plan, check:
- [ ] RLS policies defined for any new tables
- [ ] Foreign key `ON DELETE` behavior specified
- [ ] Index strategy for new query patterns
- [ ] Application code changes needed during dual-write period
