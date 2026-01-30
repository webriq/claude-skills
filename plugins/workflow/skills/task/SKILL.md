---
name: task
description: Create task documents for new features, updates, or fixes. Creates detailed specs in docs/task/*.md with full implementation context. Updates TASKS.md for tracking. Use when starting any new work.
model: opus
---

# /task - Task Planning Agent

> **Model:** opus (complex planning requires advanced reasoning)

## When to Use

Invoke `/task` when:
- Starting a new feature or enhancement
- Planning a bug fix that requires multiple changes
- User says "I want to add...", "Let's implement...", "Can we build..."
- Discussing requirements before coding
- Creating implementation specs for future work

## Invocation Options

| Command | Mode | Behavior |
|---------|------|----------|
| `/task` | Manual | You control each step: implement → test → document → ship |
| `/task auto` | Automated | After task approval, all steps run autonomously |

### Auto Mode Workflow

```
/task auto → User approves task
    ↓
/implement
    ↓
/test (Playwright E2E)
    │
PASS → /document
FAIL → /implement (with test report)
    │
    ▼
/ship → PR + notify
```

**Auto mode notes:**
- **Full automation:** Runs through implement → test → document → ship
- **Test failures:** Auto-retries by sending test report back to implement agent
- **Unexpected errors:** Stops and notifies you

## Workflow

```
User Request
     ↓
1. Discuss & clarify requirements
2. Research codebase for context
3. Create docs/task/{task-name}.md
4. Add to TASKS.md under "## Planned"
5. User can /clear and start fresh
     ↓
Ready for /implement
```

## What This Skill Does

### 1. Requirements Gathering

Ask clarifying questions:
- What is the expected behavior?
- Any UI/UX preferences?
- Priority level?

### Version Impact Guidelines

Set the `Version Impact` field based on the type of change:

| Type | Default Impact | When to Use |
|------|----------------|-------------|
| `feature` | `minor` | New functionality, backwards compatible |
| `bugfix` | `patch` | Bug fixes, no new features |
| `enhancement` | `patch` | Improvements to existing features |
| `documentation` | `patch` | Doc updates only |
| `chore` | `patch` | Maintenance, refactoring |

**Override to `major`** when:
- Breaking API changes
- Database schema changes requiring migration
- Removing deprecated features
- Changes that require user action

### 2. Codebase Research

Before creating the task document:
- Check existing similar implementations
- Identify files to modify
- Find reusable components/hooks
- Note potential pitfalls

> **Note:** Specialized skills (vercel-react-best-practices, supabase-postgres-best-practices) are invoked during `/implement`, not during task planning. This keeps planning focused on requirements and architecture.

### 3. Create Task Document

**Location:** `docs/task/{task-name}.md`

**Naming:** Use kebab-case, descriptive names:
- `user-dashboard-redesign.md`
- `lead-auto-tagging.md`
- `booking-calendar-view.md`

### 4. Update TASKS.md

Add the task to the "## Planned" section with link to task document.

**If TASKS.md doesn't exist, create it first** with this structure:

```markdown
# Tasks

Task tracking for the development workflow.

---

## Planned

Tasks ready for `/implement`.

| Task | Priority | Task Doc | Created |
|------|----------|----------|---------|

---

## In Progress

| Task | Started | Task Doc | Status |
|------|---------|----------|--------|

---

## Testing

Tasks being tested via `/test`.

| Task | Task Doc | Test Report | Status |
|------|----------|-------------|--------|

---

## Approved

Tested and approved. Ready for `/document` then `/ship`.

| Task | Task Doc | Feature Doc | Test Report | Approved |
|------|----------|-------------|-------------|----------|

---

## Ready to Ship

PRs created via `/ship`. **Items stay here until `/release` is run** (even after merge).

| Task | Branch | PR | Merged | Task Doc |
|------|--------|----|--------|----------|

---

## Shipped

Released items. Only `/release` moves items here with version number.

| Task | PR | Release | Shipped |
|------|-----|---------|---------|
```

---

## Task Document Template

Create this structure in `docs/task/{task-name}.md`:

```markdown
# {Task Title}

> **Status:** PLANNED
> **Priority:** HIGH | MEDIUM | LOW
> **Type:** feature | bugfix | enhancement | documentation | chore
> **Version Impact:** minor | patch | major
> **Created:** {Date}
> **Platform:** Web
> **Automation:** manual | auto

## Overview

{2-3 sentence description of what we're building and why}

## Requirements

### Must Have
- [ ] Requirement 1
- [ ] Requirement 2

### Nice to Have
- [ ] Optional requirement

## Current State

{Description of how things work now, if applicable}

**Current Files:**
| File | Purpose |
|------|---------|
| `path/to/file.tsx` | Description |

## Proposed Solution

{Description of the implementation approach}

### Architecture

{High-level design decisions}

### File Changes

| Action | File | Description |
|--------|------|-------------|
| CREATE | `path/to/new.tsx` | New component for X |
| MODIFY | `path/to/existing.tsx` | Add Y functionality |
| DELETE | `path/to/old.tsx` | No longer needed |

## Implementation Steps

### Step 1: {Title}
{Detailed instructions with code snippets if needed}

### Step 2: {Title}
{Detailed instructions}

## Code Examples

{Include specific code changes when helpful}

```typescript
// Example of key implementation
```

## Testing Checklist

- [ ] Test case 1
- [ ] Test case 2
- [ ] Edge case handling

## Dependencies

- Required packages: {list any new deps}
- Required APIs: {list endpoints needed}
- Blocked by: {any dependencies on other tasks}

## Notes for Implementation Agent

{Any important context the /implement agent needs to know}

## Related

- Similar feature: [link to docs]
- Design reference: [link if applicable]
```

---

## TASKS.md Integration

After creating the task document, add an entry:

```markdown
## Planned

| Task | Priority | Task Doc | Created |
|------|----------|----------|---------|
| Dashboard Redesign | HIGH | [dashboard-redesign.md](docs/task/dashboard-redesign.md) | Jan 25 |
```

---

## Output Checklist

Before completing `/task`:

- [ ] Task document created in `docs/task/`
- [ ] Document has all required sections filled
- [ ] Implementation steps are clear and actionable
- [ ] File paths verified to exist (for modifications)
- [ ] Added to TASKS.md "## Planned" section
- [ ] User understands the task and approves

---

## Handoff to /implement

When planning is complete, inform the user:

### Manual Mode
```
Task document created: docs/task/{task-name}.md
Added to TASKS.md under "Planned"

To start implementation:
1. /clear (optional - start fresh session)
2. /implement {task-name}
```

### Auto Mode
When `/task auto` was invoked and user approves the task:

1. Set `Automation: auto` in the task document
2. Use Task tool to spawn `/implement {task-name}` with **model: opus**
3. The implement skill will chain to subsequent skills automatically

```
Task approved! Starting automated pipeline...
Task: {task-name}

Spawning /implement with opus model...
```

**IMPORTANT:** In auto mode, after user approves the task:
- Do NOT wait for user to invoke /implement
- Use Task tool to spawn implement agent with model: opus
- Example: `Task({ subagent_type: "general-purpose", model: "opus", prompt: "/implement {task-name}" })`
- The automation flag in the task doc controls subsequent chaining

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/implement` | After task is approved, to start coding |
| `/test` | After implementation, to verify the feature |
| `/document` | After tests pass, to create documentation |
| `/ship` | After documentation, to create PR |
| `/release` | After multiple items shipped, to create release |

**Note:** Specialized skills (vercel-react-best-practices, supabase-postgres-best-practices) are invoked during `/implement`, not during `/task`. Install them separately from their respective repos.
