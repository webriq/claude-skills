---
name: test
description: Test web implementations using Playwright MCP for E2E testing. Creates test reports in docs/testing/*.md. Supports email-based auth testing via Mailinator (registration, verification, password reset, notifications).
model: haiku
---

# /test - Web E2E Testing Agent

> **Model:** haiku (straightforward test execution)

## When to Use

Invoke `/test {task-name}` when:
- Task is in "Testing" status in TASKS.md
- Implementation is complete from `/implement`
- Ready to verify the feature works

**Example:** `/test dashboard-redesign`

## Workflow

```
/test {task-name}
       ↓
1. Read task document for requirements
2. Check Automation field (manual | auto)
3. Read implementation for context
4. Create test plan
5. Check if email testing required (see Step 5 below)
   └── If auth keywords found → MUST use Mailinator
6. Execute tests via Playwright MCP
7. If auth flow → Verify email received & confirmation works
8. Write report to docs/testing/{task-name}.md
9. Update TASKS.md with result
       ↓
┌─── Automation Mode? ───┐
│                        │
▼ Manual                 ▼ Auto
PASS → notify user       PASS → invoke /document
FAIL → notify user       FAIL → invoke /implement with test report
```

## Auto Mode Behavior

When task document has `Automation: auto`:

### On PASS
Use Task tool to spawn document agent with **model: haiku**:
```
[AUTO] Tests PASSED. Spawning /document with haiku model...
```
`Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/document {task-name}" })`

### On FAIL
Use Task tool to spawn implement agent with **model: opus** (needs advanced reasoning to fix issues):
```
[AUTO] Tests FAILED. Spawning /implement with opus model for fixes...

Issues found:
1. {Issue summary}
2. {Issue summary}

Re-running implementation with fixes...
```
`Task({ subagent_type: "general-purpose", model: "opus", prompt: "/implement {task-name} - Fix issues from test report: {summary}" })`

**Note:** The implement skill will receive the test report context and should focus on fixing the specific issues identified. After fixes, it will chain back to /test (with haiku).

**IMPORTANT:** For any task involving registration, login, magic links, or email verification:
- The test is NOT complete until email confirmation is verified via Mailinator
- See "Email-Based Authentication Testing" section for detailed steps

## Pre-Testing Setup

### 1. Read the Task Document

```
docs/task/{task-name}.md
```

Focus on:
- Requirements checklist
- Testing checklist (if provided)
- Expected behavior

### 2. Understand the Implementation

Review the files that were changed:
- What was created?
- What was modified?
- What are the entry points?

### 3. Identify Test Scenarios

Create a test plan covering:
- Happy path (main use case)
- Edge cases
- Error handling
- Cross-platform (if applicable)

### 4. Detect If Email Testing Is Required

**CRITICAL:** Scan the task document for these keywords:
- Registration / Sign up / Create account
- Magic Link / OTP / Email verification
- Email confirmation / Confirm email
- Password reset / Forgot password
- Account activation

**If ANY of these keywords appear:**
1. **MANDATORY** - Use Mailinator for email testing (see "Email-Based Authentication Testing" section below)
2. **DO NOT** mark the test as PASS without verifying:
   - Email was received in Mailinator inbox
   - Confirmation/verification link works
   - Post-confirmation redirect is correct
3. Include "Authentication Tests" section in test report

**Example detection:**
```
Task doc says: "Magic Link authentication as primary signup method"
→ This REQUIRES email testing via Mailinator
→ Test is NOT complete until email verification is confirmed
```

---

## Testing with Playwright MCP

### Available Tools

Use the Playwright MCP tools for browser automation:

```
mcp__playwright__browser_navigate    - Go to URL
mcp__playwright__browser_snapshot    - Capture accessibility snapshot
mcp__playwright__browser_click       - Click elements
mcp__playwright__browser_type        - Type text
mcp__playwright__browser_fill_form   - Fill form fields
mcp__playwright__browser_take_screenshot - Capture screenshot
mcp__playwright__browser_console_messages - Check console
mcp__playwright__browser_network_requests - Check network
mcp__playwright__browser_wait_for    - Wait for conditions
```

### Test Execution Pattern

```typescript
// 1. Navigate to the page
mcp__playwright__browser_navigate({ url: "http://localhost:3000/path" })

// 2. Take snapshot to understand page structure
mcp__playwright__browser_snapshot()

// 3. Interact with elements (use ref from snapshot)
mcp__playwright__browser_click({ ref: "button[Submit]", element: "Submit button" })

// 4. Verify results
mcp__playwright__browser_snapshot()

// 5. Check for errors
mcp__playwright__browser_console_messages({ level: "error" })
```

### Responsive Testing

For responsive web features, resize the viewport:

```typescript
mcp__playwright__browser_resize({ width: 390, height: 844 }) // iPhone 14 Pro
mcp__playwright__browser_resize({ width: 768, height: 1024 }) // iPad
```

---

## Test Report Template

Create report in `docs/testing/{task-name}.md`:

```markdown
# Test Report: {Task Name}

> **Status:** PASS | FAIL | PARTIAL
> **Tested:** {Date}
> **Task Doc:** [link](../task/{task-name}.md)

## Summary

{1-2 sentence summary of test results}

## Test Environment

- Platform: Web
- Browser: Chromium (Playwright)
- Base URL: http://localhost:3000
- Viewport: {dimensions if relevant}

## Test Results

### Requirement Tests

| # | Requirement | Status | Notes |
|---|-------------|--------|-------|
| 1 | {Requirement from task doc} | PASS/FAIL | {Details} |
| 2 | {Requirement} | PASS/FAIL | {Details} |

### Functional Tests

#### Test 1: {Test Name}
**Steps:**
1. Navigate to {page}
2. Click {element}
3. Verify {expected result}

**Result:** PASS/FAIL
**Evidence:** {Screenshot or description}

#### Test 2: {Test Name}
{Same format}

### Edge Cases

| Case | Result | Notes |
|------|--------|-------|
| Empty state | PASS/FAIL | {Details} |
| Error handling | PASS/FAIL | {Details} |
| Loading state | PASS/FAIL | {Details} |

### Console Errors

```
{Any console errors found, or "None"}
```

### Network Issues

```
{Any failed requests, or "None"}
```

## Issues Found

### Issue 1: {Title}
**Severity:** Critical | Major | Minor
**Description:** {What's wrong}
**Steps to Reproduce:**
1. Step 1
2. Step 2
**Expected:** {What should happen}
**Actual:** {What actually happens}
**Screenshot:** {If applicable}

### Issue 2: {Title}
{Same format}

## Screenshots

{Include relevant screenshots}

## Recommendations

{Any suggestions for fixes or improvements}

## Verdict

**PASS** - All requirements met, ready for documentation
OR
**FAIL** - Issues found, needs fixes (see Issues section)
OR
**PARTIAL** - Core functionality works, minor issues noted
```

---

## Update TASKS.md

### If PASS

Move to "Approved" section (pending user approval):

```markdown
## Testing

| Task | Task Doc | Test Report | Status |
|------|----------|-------------|--------|
| Quick Actions Redesign | [link](...) | [report](docs/testing/...) | PASS - Awaiting approval |
```

### If FAIL

Keep in "Testing" with failure note:

```markdown
## Testing

| Task | Task Doc | Test Report | Status |
|------|----------|-------------|--------|
| Quick Actions Redesign | [link](...) | [report](docs/testing/...) | FAIL - See report |
```

---

## Handoff

**Check the task document for `Automation: auto` field.**

### Manual Mode

#### On PASS
```
Testing complete: {task-name}

Result: PASS
Test Report: docs/testing/{task-name}.md

All requirements verified. Ready for your approval.
Once approved, run: /document {task-name}
```

#### On FAIL
```
Testing complete: {task-name}

Result: FAIL
Test Report: docs/testing/{task-name}.md

Issues found:
1. {Issue summary 1}
2. {Issue summary 2}

To fix and re-test:
1. /implement {task-name} (review test report for fixes)
2. /test {task-name} (re-run tests)
```

### Auto Mode

#### On PASS
```
Testing complete: {task-name}

Result: PASS
Test Report: docs/testing/{task-name}.md

[AUTO] Spawning /document with haiku model...
```
Use Task tool: `Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/document {task-name}" })`

#### On FAIL
```
Testing complete: {task-name}

Result: FAIL
Test Report: docs/testing/{task-name}.md

Issues found:
1. {Issue summary 1}
2. {Issue summary 2}

[AUTO] Spawning /implement with opus model for fixes...
```
Use Task tool: `Task({ subagent_type: "general-purpose", model: "opus", prompt: "/implement {task-name} - Fix: {issue summaries}" })`

---

## Email-Based Authentication Testing

For testing account registration, email verification, password reset, and email notifications using temporary email services.

### Supported Email Services

| Service | Domain | Inbox URL | Best For |
|---------|--------|-----------|----------|
| **Mailinator** | `@mailinator.com` | `mailinator.com/v4/public/inboxes.jsp?to={name}` | Primary choice - reliable, no signup |
| **Guerrilla Mail** | Dynamic | `guerrillamail.com` | Alternative if Mailinator blocked |
| **TempMail** | Dynamic | `temp-mail.org` | Backup option |

### Email Testing Workflow

```
1. Generate unique test email
2. Navigate to registration page
3. Fill registration form with test email
4. Navigate to email inbox
5. Wait for & extract verification link
6. Complete verification
7. Verify account is active
```

### Generating Test Emails

Use a unique identifier to avoid inbox collisions:

```typescript
// Pattern: test-{timestamp}-{random}@mailinator.com
const testEmail = `test-${Date.now()}-${Math.random().toString(36).slice(2, 6)}@mailinator.com`;
// Example: test-1706234567890-x7k2@mailinator.com
```

### Step-by-Step: Account Registration Test

#### Step 1: Navigate to Registration Page

```typescript
mcp__playwright__browser_navigate({ url: "http://localhost:3000/register" })
mcp__playwright__browser_snapshot()
```

#### Step 2: Fill Registration Form

```typescript
// Get element refs from snapshot, then fill form
mcp__playwright__browser_fill_form({
  fields: [
    { name: "Email", type: "textbox", ref: "input[email]", value: testEmail },
    { name: "Password", type: "textbox", ref: "input[password]", value: "TestPassword123!" },
    { name: "Confirm Password", type: "textbox", ref: "input[confirmPassword]", value: "TestPassword123!" }
  ]
})
mcp__playwright__browser_click({ ref: "button[Submit]", element: "Submit button" })
```

#### Step 3: Navigate to Mailinator Inbox

```typescript
// Extract the inbox name from email (part before @)
const inboxName = testEmail.split('@')[0];
mcp__playwright__browser_navigate({
  url: `https://www.mailinator.com/v4/public/inboxes.jsp?to=${inboxName}`
})
```

#### Step 4: Wait for Email & Open It

```typescript
// Wait for email to arrive (may take a few seconds)
mcp__playwright__browser_wait_for({ time: 5 })
mcp__playwright__browser_snapshot()

// Click on the email row (look for email from your app)
mcp__playwright__browser_click({ ref: "row[YourApp]", element: "Email from YourApp" })
mcp__playwright__browser_snapshot()
```

#### Step 5: Extract Verification Link

```typescript
// The email content loads in an iframe - take snapshot to see content
mcp__playwright__browser_snapshot()

// Look for verification link in the email body
// The link format depends on your auth provider (Supabase: /auth/confirm?token=...)
// Click the verification link or extract URL

// Option A: Click link directly if visible
mcp__playwright__browser_click({ ref: "link[Verify]", element: "Verification link" })

// Option B: Use evaluate to extract href if needed
mcp__playwright__browser_evaluate({
  function: "() => { return document.querySelector('a[href*=\"confirm\"]')?.href }"
})
```

#### Step 6: Complete Verification

```typescript
// After clicking verification link, verify success
mcp__playwright__browser_snapshot()
// Should see success message or redirect to dashboard
```

### Password Reset Testing

```typescript
// 1. Navigate to forgot password
mcp__playwright__browser_navigate({ url: "http://localhost:3000/forgot-password" })
mcp__playwright__browser_snapshot()

// 2. Enter test email
mcp__playwright__browser_type({ ref: "input[email]", text: testEmail })
mcp__playwright__browser_click({ ref: "button[Reset]", element: "Reset password button" })

// 3. Check Mailinator for reset email
mcp__playwright__browser_navigate({
  url: `https://www.mailinator.com/v4/public/inboxes.jsp?to=${inboxName}`
})
mcp__playwright__browser_wait_for({ time: 5 })
mcp__playwright__browser_snapshot()

// 4. Open reset email and click link
mcp__playwright__browser_click({ ref: "row[Password Reset]", element: "Password reset email" })
mcp__playwright__browser_snapshot()

// 5. Extract and navigate to reset link
// 6. Set new password and verify
```

### Email Notification Testing

For testing transactional emails (booking confirmations, notifications, etc.):

```typescript
// 1. Trigger the action that sends email (e.g., make a booking)
mcp__playwright__browser_navigate({ url: "http://localhost:3000/book" })
// ... complete booking flow ...

// 2. Check inbox for notification
mcp__playwright__browser_navigate({
  url: `https://www.mailinator.com/v4/public/inboxes.jsp?to=${inboxName}`
})
mcp__playwright__browser_wait_for({ time: 10 }) // Notifications may take longer
mcp__playwright__browser_snapshot()

// 3. Verify email content
mcp__playwright__browser_click({ ref: "row[Booking Confirmation]", element: "Confirmation email" })
mcp__playwright__browser_snapshot()

// 4. Validate email contains expected content
// - Booking details
// - Correct dates/times
// - Links work
```

### Handling Mailinator's Interface

Mailinator's UI structure:

```
Page Structure:
├── Search/Filter bar
├── Inbox table
│   ├── Row 1: [From] [Subject] [Time]
│   ├── Row 2: [From] [Subject] [Time]
│   └── ...
└── Email content iframe (when email selected)
```

**Tips:**
- Take snapshot after navigation to get current refs
- Emails appear in reverse chronological order (newest first)
- Email content loads in an iframe - may need to interact with iframe
- If inbox is empty, wait and refresh

### Troubleshooting

| Issue | Solution |
|-------|----------|
| Email not arriving | Wait longer (up to 30s), check spam folder in Mailinator |
| Mailinator blocked | Use Guerrilla Mail or TempMail as alternative |
| Can't click email link | Extract URL with `browser_evaluate` and navigate directly |
| Wrong inbox | Verify inbox name matches email prefix exactly |
| Supabase rate limiting | Use different test emails, wait between tests |

### Test Report: Email Authentication Section

Add to test report when testing auth flows:

```markdown
### Authentication Tests

| Test | Email Used | Result | Notes |
|------|------------|--------|-------|
| Registration | test-xxx@mailinator.com | PASS/FAIL | {Details} |
| Email verification | (same) | PASS/FAIL | Link received in Xs |
| Password reset | (same) | PASS/FAIL | {Details} |
| Login after verify | (same) | PASS/FAIL | {Details} |

### Email Delivery

| Email Type | Received | Time | Content Valid |
|------------|----------|------|---------------|
| Verification | Yes/No | Xs | Yes/No |
| Password Reset | Yes/No | Xs | Yes/No |
| Booking Confirm | Yes/No | Xs | Yes/No |
```

---

## Manual Testing Guidance

For features that can't be fully automated:

### What to Test Manually

- Complex user interactions
- Visual design accuracy
- Animation smoothness

### Document Manual Tests

Include in report:

```markdown
## Manual Testing Required

| Test | Instructions | Expected Result |
|------|--------------|-----------------|
| Visual accuracy | Compare to design | Matches mockup |
| Animation | Trigger X action | Smooth 60fps |
```

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/implement` | When tests fail, return to implement |
| `/document` | After tests pass and user approves |
| `/react-best-practices` | React/Next.js patterns during debugging |
| `/postgres-best-practices` | Database-related test issues |
