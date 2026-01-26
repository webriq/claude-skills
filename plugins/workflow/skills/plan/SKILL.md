---
name: plan
description: Plan new features, updates, or fixes. Creates detailed task documents in docs/task/*.md with full implementation context. Updates TASKS.md for tracking. Use when starting any new work.
model: opus
---

# /plan - Task Planning Agent

> **Model:** opus (complex planning requires advanced reasoning)

## When to Use

Invoke `/plan` when:
- Starting a new feature or enhancement
- Planning a bug fix that requires multiple changes
- User says "I want to add...", "Let's implement...", "Can we build..."
- Discussing requirements before coding
- Creating implementation specs for future work

## Invocation Options

| Command | Mode | Behavior |
|---------|------|----------|
| `/plan` | Manual | You control each step: implement → test → document → ship |
| `/plan auto` | Automated | After plan approval, all steps run autonomously |

### Auto Mode Workflow

```
/plan auto → User approves plan
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

**Invoke specialized skills as needed:**
- `/react-best-practices` - For React/Next.js optimization patterns
- `/postgres-best-practices` - For database design and queries

### 3. Create Task Document

**Location:** `docs/task/{task-name}.md`

**Naming:** Use kebab-case, descriptive names:
- `user-dashboard-redesign.md`
- `lead-auto-tagging.md`
- `booking-calendar-view.md`

### 4. Update TASKS.md

Add the task to the "## Planned" section with link to task document.

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

Before completing `/plan`:

- [ ] Task document created in `docs/task/`
- [ ] Document has all required sections filled
- [ ] Implementation steps are clear and actionable
- [ ] File paths verified to exist (for modifications)
- [ ] Added to TASKS.md "## Planned" section
- [ ] User understands the plan and approves

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
When `/plan auto` was invoked and user approves the plan:

1. Set `Automation: auto` in the task document
2. Use Task tool to spawn `/implement {task-name}` with **model: opus**
3. The implement skill will chain to subsequent skills automatically

```
Plan approved! Starting automated pipeline...
Task: {task-name}

Spawning /implement with opus model...
```

**IMPORTANT:** In auto mode, after user approves the plan:
- Do NOT wait for user to invoke /implement
- Use Task tool to spawn implement agent with model: opus
- Example: `Task({ subagent_type: "general-purpose", model: "opus", prompt: "/implement {task-name}" })`
- The automation flag in the task doc controls subsequent chaining

---

## Related Skills

| Skill | When to Invoke |
|-------|----------------|
| `/react-best-practices` | React/Next.js performance patterns |
| `/postgres-best-practices` | Database design, queries, RLS policies |
