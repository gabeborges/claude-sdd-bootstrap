---
name: debug:investigate
description: Systematic root-cause analysis for failures — reproduce, trace, narrow, fix ticket
---

## Required Reading — Do This First

Before starting the investigation, read this file completely:

1. `.claude/skills/debugging/SKILL.md` — methodology, fix ticket format, common failure patterns

Do not skip this. The debugging methodology is in this file.

---

## Scope

Systematic root-cause analysis for failures in code, tests, or runtime behavior.

**Use for:**
- Bug investigation
- Test failures
- Runtime errors
- Performance issues
- Integration failures
- Spec mismatches

**Not for:**
- Feature implementation (use appropriate SDD agent)
- Code review (use `code-reviewer` agent)
- Security audits (use `/security:review`)

---

## Arguments

`$ARGUMENTS` should describe the failure:
- **Error message** (e.g., "TypeError: Cannot read property 'name' of undefined")
- **Failing test** (e.g., "Test: User profile update returns 500")
- **Bug report** (e.g., "Dashboard shows stale data after account switch")
- **File/line reference** (e.g., "app/api/users/route.ts:45 throws 403")

If no arguments, ask user to describe the failure.

---

## Investigation Process

Follow the 4-step methodology from `debugging/SKILL.md`:

### 1. Reproduce

**Goal:** Establish reliable reproduction steps.

#### What to gather:
- Exact steps to trigger the failure
- Environment details (browser, OS, data state, user role, timing)
- Failure rate (100% consistent or intermittent?)
- Minimal reproduction case (simplest way to trigger)

#### Questions to answer:
- Does it fail every time, or only under certain conditions?
- What's different between working and failing cases?
- Can you reproduce it in isolation (minimal test case)?

#### Output:
Create a minimal reproduction section:
```markdown
## Reproduction Steps

1. [Step 1]
2. [Step 2]
3. [Step 3]
4. **Observe**: [Expected vs. Actual behavior]

**Failure rate**: [100% / Intermittent]
**Environment**: [Browser, OS, data state, user role]
```

### 2. Trace

**Goal:** Follow execution path from entry point to failure.

#### What to do:
- Read the relevant code files
- Identify the execution path that leads to the error
- Add logging at decision points if needed (mentally or in code)
- Distinguish symptom (what user sees) from cause (where code breaks)

#### Use these tools:
- Read files along execution path
- Grep for function calls, imports, references
- Check recent changes: `git blame`, `git log -- <file>`
- Look for related files (tests, types, related components)

#### Output:
Document the execution trace:
```markdown
## Execution Trace

1. Entry: [File/function where request enters]
2. Path: [Key functions/components called]
3. Failure point: [Exact file:line where error occurs]
4. Root cause: [The condition/logic error causing failure]
```

### 3. Narrow

**Goal:** Isolate the minimal conditions required for failure.

#### What to test:
- Boundary cases: `null`, `undefined`, empty arrays, `0`, missing fields
- Recent changes: What changed that could affect this?
- Related features: Does the same pattern fail elsewhere?
- Minimal condition set: What's the smallest change that fixes it?

#### Common failure patterns to check:

**Timing Issues:**
- Race conditions (async ops out of order)
- Missing awaits
- Event handlers firing before state ready

**Null/Undefined:**
- Undefined property access (`obj.field.nested` when `field` is undefined)
- Null returns
- Missing optional chaining (`?.`)

**Async Errors:**
- Unhandled promise rejections
- Empty catch blocks
- No timeout handling

**State Management:**
- Stale closures
- Direct state mutation
- Missing dependencies in useEffect/useCallback

**Type Coercion:**
- String vs number (`"10" + 5` = `"105"`)
- Truthy/falsy confusion
- Array/object comparison

**Data Assumptions:**
- Missing validation
- Empty states not handled
- Edge cases (zero, negative, very large, special chars)

#### Output:
```markdown
## Isolation

**Root cause**: [Exact condition causing failure]
**File:Line**: `{file}:{line}`
**Pattern**: [Which failure pattern from above]
**Evidence**: [Code snippet showing the issue]
```

### 4. Verify

**Goal:** Confirm diagnosis and assess fix impact.

#### What to verify:
- Does your diagnosis explain the failure?
- Will the proposed fix resolve it?
- What tests currently cover this code path?
- What else could this fix affect?

#### Regression risk assessment:

**What else uses this code?**
- Grep for imports and function calls
- Check test files for coverage
- Look for similar patterns elsewhere

**What assumptions might break?**
- Does this affect API contracts?
- Are there implicit dependencies?
- Could this alter timing/order/state?

**Risk level:**
- **Low**: Isolated change, well-tested, no shared dependencies
- **Medium**: Shared utilities, API behavior, multiple features
- **High**: Core infrastructure, widely-used, data layer, auth

#### Output:
```markdown
## Verification

**Diagnosis confirmed**: [Yes/No + evidence]
**Proposed fix**: [Specific change to make]
**Regression risk**: [Low/Medium/High + justification]
**Tests to verify**: [Existing tests that should pass]
**New tests needed**: [Edge cases to cover]
```

---

## Fix Ticket Format

After completing the investigation, generate a fix ticket using this format:

```markdown
### T-{NNN}: Fix: {precise description of what's broken}

**root-cause**: `{file}:{line}` — {exact condition causing failure}
**implements**: `{spec node or feature reference}`
**repro**:
1. [Step 1]
2. [Step 2]
3. **Observe**: [Failure description]

**fix-approach**: {minimal, testable fix}
**regression-risk**: {what else could break; tests to verify}
```

### Fix Ticket Rules

**Must include:**
- Exact file and line number
- Specific condition or logic error
- Minimal reproduction steps (3-5 max)
- Proposed fix that's minimal and testable
- Regression risk assessment

**Must NOT:**
- Create vague "investigate X" tickets (must have root cause identified)
- Propose broad refactors when targeted fix works
- Skip regression risk assessment
- Guess at cause without evidence

---

## Example Investigation

### Input

`$ARGUMENTS`: "Dashboard metrics show stale data after switching accounts"

### Investigation

#### 1. Reproduce

```markdown
## Reproduction Steps

1. Log in as User A, wait for metrics to load
2. Switch to User B using account switcher
3. **Observe**: Dashboard shows User A's metrics for 2-3 seconds before updating

**Failure rate**: 100% reproducible
**Environment**: Chrome 120, macOS, any users with different metric values
```

#### 2. Trace

Read `app/dashboard/Dashboard.tsx`:
- Component has `useEffect` that fetches metrics on mount
- Dependency array includes `userId`
- No cleanup function in useEffect

```markdown
## Execution Trace

1. Entry: `Dashboard.tsx` component mounts
2. useEffect triggers metric fetch when `userId` changes
3. User switches accounts → `userId` updates
4. New request starts, but previous request still in flight
5. Previous request completes AFTER new request → overwrites fresh data with stale

**Failure point**: `app/dashboard/Dashboard.tsx:34` — useEffect doesn't cancel in-flight requests
**Root cause**: Race condition — old requests complete after new ones, overwriting fresh data
```

#### 3. Narrow

Tested:
- Slow account switch (>5 seconds) → No issue (old request completes first)
- Fast account switch (<1 second) → Issue occurs (race condition)
- Pattern: Classic async cleanup missing

```markdown
## Isolation

**Root cause**: useEffect cleanup doesn't cancel previous requests
**File:Line**: `app/dashboard/Dashboard.tsx:34`
**Pattern**: Async Errors — missing cleanup for in-flight requests
**Evidence**:
  ```typescript
  useEffect(() => {
    async function fetchMetrics() {
      const data = await supabase
        .from('metrics')
        .select('*')
        .eq('user_id', userId)
      setMetrics(data)
    }
    fetchMetrics()
    // Missing: return () => cleanup
  }, [userId])
  ```
```

#### 4. Verify

```markdown
## Verification

**Diagnosis confirmed**: Yes — rapid account switching triggers race condition
**Proposed fix**: Add AbortController to cancel previous requests:
  ```typescript
  useEffect(() => {
    const controller = new AbortController()

    async function fetchMetrics() {
      const data = await supabase
        .from('metrics')
        .select('*')
        .eq('user_id', userId)
        .abortSignal(controller.signal)
      setMetrics(data)
    }

    fetchMetrics()

    return () => controller.abort()
  }, [userId])
  ```

**Regression risk**: Medium — affects all dashboard metric fetching
- Verify: Cancellation doesn't break loading states
- Test: Rapid account switching multiple times
- Check: Error boundaries handle aborted requests gracefully
- Search: Any other useEffects fetching data without cleanup

**Tests to verify**:
- Existing: `Dashboard.test.tsx` — metrics load correctly
- New: Test rapid account switching (race condition)

**New tests needed**: Edge case for aborted requests
```

#### Fix Ticket

```markdown
### T-012: Fix: Cancel in-flight metric requests when user switches accounts

**root-cause**: `app/dashboard/Dashboard.tsx:34` — useEffect fetches metrics but doesn't cancel previous requests in cleanup; race condition allows stale responses to overwrite fresh data

**implements**: Dashboard should show current user's data (core requirement)

**repro**:
1. Log in as any user, wait for metrics to load
2. Switch accounts quickly (<1 second)
3. **Observe**: Brief flash of previous user's data before correct data appears

**fix-approach**: Add AbortController to useEffect, call `controller.abort()` in cleanup function before fetching new data

**regression-risk**: Medium — affects all dashboard metric fetching; verify cancellation doesn't break loading states; test rapid account switching; ensure error boundaries handle aborted requests gracefully
```

---

## Escalation Points

### Create spec-change-request when:
- Implementation is correct but spec is wrong
- Spec is ambiguous (different interpretations possible)
- Compliance/security conflicts with spec

### Stop and ask when:
- Root cause spans multiple features or build versions
- Fix requires architectural changes
- Multiple conflicting requirements
- Data loss or security implications unclear

### Do NOT:
- Modify specs to match broken behavior
- Make architectural decisions without approval
- Implement security workarounds

---

## Special Cases

### Test Failure

If investigating a test failure:
1. Read test file and understand what's being tested
2. Run test in isolation to verify failure
3. Compare test expectations vs. actual behavior
4. Determine if test is wrong or code is wrong
5. If test is wrong, update test; if code is wrong, create fix ticket

### Spec Mismatch

If implementation doesn't match spec:
1. Verify what spec says
2. Verify what code does
3. Determine if spec is correct or needs update
4. If spec is wrong: create `spec-change-request.yaml` entry
5. If code is wrong: create fix ticket referencing spec

### Performance Issue

If investigating performance:
1. Establish baseline (how slow? under what conditions?)
2. Profile the code (use browser DevTools, React Profiler, logs)
3. Identify bottleneck (rendering, network, computation)
4. Propose targeted optimization
5. Assess risk of optimization (complexity vs. benefit)

---

## Output Format

Always produce:

1. **Investigation Summary** (concise overview)
2. **Reproduction Steps** (minimal case)
3. **Execution Trace** (path to failure)
4. **Isolation** (root cause identified)
5. **Verification** (diagnosis confirmed, fix proposed)
6. **Fix Ticket** (standard format from `debugging/SKILL.md`)

---

## Communication Style

- Be systematic and methodical
- Show your work (trace, evidence, reasoning)
- Reference specific files and line numbers
- Provide code snippets for clarity
- Use the 4-step framework explicitly
- No guessing — if you need more info, ask
- Be precise about root cause vs. symptom
