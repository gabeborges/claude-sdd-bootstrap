---
name: debugging
description: Root-cause analysis methodology, fix ticket format, and common failure patterns for debugging and incident resolution
---

# Debugging

Systematic approach to investigating failures, narrowing to root cause, and producing actionable fix tickets.

## Scope

**Use for:** Bug investigation, test failures, runtime errors, performance issues, integration failures

**Not for:** Direct code fixes (create tickets instead), feature enhancements, refactoring

---

# Root-Cause Analysis Methodology

## The Process: Reproduce → Trace → Narrow → Verify

**1. Reproduce**
- Get the exact steps to trigger the failure
- Establish baseline: does it fail 100% of the time, or intermittently?
- Document the environment: browser, OS, data state, user role, timing
- Create a minimal reproduction case — simplest possible way to trigger the issue

**2. Trace**
- Follow the execution path from entry point to failure
- Add logging at decision points if needed
- Identify the exact line where things go wrong
- Distinguish between symptom (what the user sees) and cause (where the code breaks)

**3. Narrow**
- Remove variables until you isolate the condition
- Test boundary cases: null, undefined, empty arrays, zero, missing fields
- Check recent changes: git blame, recent PRs, related commits
- Identify the minimal set of conditions required for the failure

**4. Verify**
- Confirm your diagnosis with a test case
- Verify the proposed fix resolves the issue without breaking other tests
- Document regression risk: what else could this change affect?

---

# Fix Ticket Format

Every fix ticket must include:

```markdown
### T-{NNN}: Fix: {precise description of what's broken}

**root-cause**: {exact file:line and condition that causes failure}
**implements**: `{original spec node or feature reference}`
**repro**: {minimal steps to reproduce}
**fix-approach**: {suggested fix — minimal and verifiable}
**regression-risk**: {what else could break; related tests to verify}
```

## Rules for Fix Tickets

**Must include:**
- Exact file and line number where the issue occurs
- The specific condition or logic error causing the failure
- Minimal reproduction steps (3-5 steps max)
- A proposed fix that's minimal and testable
- Assessment of what else might break

**Must NOT:**
- Create vague "investigate X" tickets (must have identified root cause)
- Propose broad refactors when a targeted fix will do
- Skip the regression risk assessment
- Guess at the cause without evidence

---

# Regression Risk Assessment

Before creating a fix ticket, answer:

**What else uses this code?**
- Grep for imports and function calls
- Check test files for coverage
- Look for similar patterns elsewhere in the codebase

**What assumptions might break?**
- Does this change affect API contracts?
- Are there implicit dependencies on the current behavior?
- Could this alter timing, order, or state in other flows?

**How can we verify the fix is safe?**
- Which existing tests cover this code path?
- Do we need new tests for the edge case?
- Should we add regression tests for this specific failure?

**Risk levels:**
- **Low**: Isolated change, well-tested code path, no shared dependencies
- **Medium**: Touches shared utilities, changes API behavior, affects multiple features
- **High**: Core infrastructure, widely-used functions, data layer changes, auth/permissions

---

# Common Failure Patterns

## Timing Issues
- **Race conditions**: Async operations completing out of order
- **Missing awaits**: Promises not resolved before use
- **Event handlers**: Firing before state is ready
- **Fix approach**: Add proper async/await, use loading states, ensure sequential execution

## Null/Undefined
- **Undefined property access**: `obj.field.nested` when `field` is undefined
- **Null returns**: Functions returning null instead of expected objects
- **Optional chaining**: Missing `?.` when accessing nested properties
- **Fix approach**: Add existence checks, use optional chaining, provide defaults

## Async Errors
- **Unhandled promise rejections**: Missing .catch() or try/catch
- **Error swallowing**: Empty catch blocks hiding failures
- **Timeout issues**: No timeout handling on network requests
- **Fix approach**: Add proper error boundaries, log errors, set timeouts

## State Management
- **Stale closures**: Closures capturing old state values
- **State mutation**: Directly modifying state instead of creating new objects
- **Missing dependencies**: useEffect/useCallback with incomplete dependency arrays
- **Fix approach**: Use functional updates, immutable patterns, complete dependencies

## Type Coercion
- **String vs number**: `"10" + 5` = `"105"` instead of `15`
- **Truthy/falsy confusion**: `0`, `""`, `null`, `undefined` all falsy
- **Array/object comparison**: `[] === []` is false
- **Fix approach**: Strict equality (===), explicit type conversion, type guards

## Data Assumptions
- **Missing validation**: Assuming data shape without validation
- **Empty states**: Not handling empty arrays, no results, missing fields
- **Edge cases**: Zero, negative numbers, very large values, special characters
- **Fix approach**: Add input validation, handle empty states, test boundaries

---

# Minimal Reproduction

A minimal repro is the smallest possible set of steps that triggers the failure.

**Good minimal repro:**
```
1. Log in as non-admin user
2. Navigate to /settings
3. Click "Update Profile"
4. Observe: 403 error instead of form
```

**Bad repro (not minimal):**
```
1. Create new account
2. Verify email
3. Add profile photo
4. Set up 2FA
5. Navigate to settings
6. Fill out entire profile form
7. Submit
8. Go back and try to edit
9. Click update
10. Error appears sometimes
```

**Making it minimal:**
- Remove steps that don't affect the outcome
- Use existing test data instead of creating new data
- Focus on the exact action that triggers the failure
- Note if it's intermittent (and what makes it intermittent)

---

# Escalation Points

**Create spec-change-request when:**
- The implementation is correct but the spec is wrong
- The spec is ambiguous and different interpretations lead to different behavior
- Compliance or security requirements conflict with the spec

**Stop and ask when:**
- Root cause spans multiple features or build versions
- The fix requires architectural changes
- Multiple conflicting requirements exist
- Data loss or security implications are unclear

**Do NOT:**
- Modify specs to match broken behavior
- Make architectural decisions without approval
- Implement workarounds that bypass security controls

---

# Examples

## Example 1: Pagination Bug

**Failure report:** "GET /users returns `nextCursor: null` instead of omitting the field when there are no more pages"

**Root-cause analysis:**
1. **Reproduce**: GET /users with limit=50, only 10 users exist → response includes `"nextCursor": null`
2. **Trace**: Follow code from route → handler → buildResponse()
3. **Narrow**: buildResponse() always includes nextCursor in object spread, even when undefined (serializes to null)
4. **Verify**: Confirmed spec says "omit field when no more pages", not "return null"

**Fix ticket:**
```markdown
### T-004: Fix: Omit nextCursor when no more pages instead of returning null

**root-cause**: `src/routes/users.ts:45` — `buildResponse()` always includes nextCursor in spread, even when undefined (serialized as null)
**implements**: `/paths/users/get` (spec requires field omission, not null)
**repro**: GET /users with fewer results than page size → response includes `"nextCursor": null`
**fix-approach**: Use conditional spread: `...(nextCursor && { nextCursor })` in buildResponse
**regression-risk**: Low — only affects empty pagination case; existing cursor-based pagination tests unaffected; verify all paginated endpoints handle empty next cursor correctly
```

## Example 2: React State Bug

**Failure report:** "Dashboard metrics show stale data after user switches accounts"

**Root-cause analysis:**
1. **Reproduce**: Log in as User A, switch to User B, metrics still show User A's data for 2-3 seconds
2. **Trace**: useEffect in Dashboard.tsx fetches metrics on mount, dependency array includes `userId`
3. **Narrow**: State update happens, but cleanup function doesn't cancel in-flight requests
4. **Verify**: Race condition — User A's request completes after User B's request starts, overwriting fresh data with stale

**Fix ticket:**
```markdown
### T-012: Fix: Cancel in-flight metric requests when user switches accounts

**root-cause**: `app/dashboard/Dashboard.tsx:34` — useEffect fetches metrics but doesn't cancel previous requests in cleanup; race condition allows stale responses to overwrite fresh data
**implements**: Dashboard should show current user's data (core requirement)
**repro**:
1. Log in as any user, wait for metrics to load
2. Switch accounts quickly
3. Observe brief flash of previous user's data

**fix-approach**: Add AbortController to useEffect, call controller.abort() in cleanup function before fetching new data
**regression-risk**: Medium — affects all dashboard metric fetching; verify cancellation doesn't break loading states; test rapid account switching; ensure error boundaries handle aborted requests gracefully
```

## Example 3: Validation Edge Case

**Failure report:** "Form allows submission with spaces-only name field"

**Root-cause analysis:**
1. **Reproduce**: Enter "   " (only spaces) in name field, form submits successfully
2. **Trace**: Validation checks `name.length > 0`, which passes for spaces
3. **Narrow**: No trim() applied before validation
4. **Verify**: Backend also doesn't trim, so "   " is stored in database

**Fix ticket:**
```markdown
### T-008: Fix: Reject spaces-only input in name validation

**root-cause**: `app/components/ProfileForm.tsx:56` — validation checks `name.length > 0` without trimming, allowing spaces-only strings
**implements**: Form validation requirements (names must contain non-whitespace characters)
**repro**: Enter only spaces in name field → form submits with empty-looking name
**fix-approach**: Change validation to `name.trim().length > 0`; also trim on backend before storage
**regression-risk**: Low — purely additive validation; existing valid names unaffected; verify trim doesn't break multi-word names or legitimate whitespace
```

---

# Integration with SDD

- **Read specs first**: Understand expected behavior before diagnosing
- **Check tasks.yaml**: See if similar issues were addressed before
- **Reference spec nodes**: Fix tickets must point back to original requirements
- **Update checks.yaml**: Log investigation findings for future reference
- **Create spec-change-requests**: When specs conflict with reality or compliance

---

# Commands

- `/debugging:analyze` — Start root-cause analysis on reported failure
- `/debugging:repro` — Create minimal reproduction steps from verbose report
- `/debugging:ticket` — Generate fix ticket from diagnosed issue
