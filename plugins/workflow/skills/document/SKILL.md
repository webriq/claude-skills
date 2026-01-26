---
name: document
description: Update project documentation after feature approval. Creates/updates feature docs and user guides. Use after /test passes and user approves.
model: haiku
---

# /document - Documentation Agent

> **Model:** haiku (templated documentation work)

## When to Use

Invoke `/document {task-name}` when:
- Task has passed testing (`/test` returned PASS)
- User has approved the implementation
- Ready to update project documentation

**Example:** `/document dashboard-redesign`

## Workflow

```
/document {task-name}
       ↓
1. Read task document for context
2. Check Automation field (manual | auto)
3. Read test report for verification
4. Update/create feature documentation
5. Update/create user guide (if user-facing)
6. Update CLAUDE.md files if needed
7. Move task to "Approved" in TASKS.md
       ↓
┌─── Automation Mode? ───┐
│                        │
▼ Manual                 ▼ Auto
Notify user              Invoke /ship {task-name}
Ready for /ship
```

## Auto Mode Behavior

When task document has `Automation: auto`:

After documentation is complete, use Task tool to spawn ship agent with **model: haiku**:
```
Documentation complete: {task-name}

Updated files:
- docs/features/{feature}.md
- docs/guides/{feature}.md

[AUTO] Spawning /ship with haiku model...
```
`Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/ship {task-name}" })`

## Pre-Documentation Checklist

### 1. Gather Context

Read these files:
```
docs/task/{task-name}.md        - Implementation details
docs/testing/{task-name}.md     - Test results & verification
```

### 2. Identify Documentation Needs

| Change Type | Documentation Needed |
|-------------|---------------------|
| New feature | Feature doc + User guide |
| Enhancement | Update existing docs |
| Bug fix | Update troubleshooting sections |
| API change | Update API reference |

### 3. Use Templates

Always follow templates in `docs/templates/`:
- `FEATURE_TEMPLATE.md` - Technical feature docs
- `USER_GUIDE_TEMPLATE.md` - User-facing guides

---

## Documentation Types

### 1. Feature Documentation

**Location:** `docs/features/{FEATURE}.md`
**Audience:** Developers
**Purpose:** Technical implementation details

**When to create/update:**
- New feature → Create new doc
- Enhancement → Update existing doc
- Significant change → Update relevant sections

### 2. User Guide

**Location:** `docs/guides/{feature}.md`
**Audience:** End users (business owners, customers)
**Purpose:** How to use the feature

**When to create/update:**
- User-facing feature → Create/update guide
- UI changes → Update screenshots/steps
- New functionality → Add new sections

### 3. CLAUDE.md Files

**Locations:**
- `/CLAUDE.md` - Root project instructions

**When to update:**
- New patterns discovered
- New "Do Not" rules
- Bug fix patterns
- Tech stack changes

---

## Feature Documentation Structure

```markdown
# Feature: {Feature Name}

> **Status:** PRODUCTION
> **Last Updated:** {Date}

## Overview

{Brief 2-3 sentence description}

---

## User Journey

### For Customers
1. Step 1
2. Step 2

### For Business Owners
1. Step 1
2. Step 2

---

## Architecture

### File Structure
{Accurate file tree - VERIFY paths exist}

### Database Schema
{SQL with comments, if applicable}

### API Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|

---

## Implementation Details

### Key Components
| Component | Location | Purpose |
|-----------|----------|---------|

### Technical Notes
- Important detail 1
- Important detail 2

---

## Related Features
- [Link to related feature]
```

**Target length:** 200-400 lines

---

## User Guide Structure

```markdown
# {Feature Name} Guide

> {Brief intro sentence}

## Quick Start
{1-2 paragraph overview}

---

## For Customers

### How to {Primary Action}
1. Step with context
2. Step with context

### Tips
- Tip 1
- Tip 2

---

## For Business Owners

### How to {Admin Action}
1. Step with context

### Settings
| Setting | Location | What it does |
|---------|----------|--------------|

---

## FAQ

**Q: Question?**
A: Answer.

---

## Troubleshooting

**Issue:** Description
**Solution:** How to fix

---

## Related Guides
- [Link]
```

**Target length:** 100-200 lines

---

## Documentation Checklist

### Feature Doc
- [ ] Follows template structure
- [ ] File paths verified to exist
- [ ] API endpoints are accurate
- [ ] Schema matches database
- [ ] Status set to PRODUCTION
- [ ] Last Updated date set
- [ ] Under 400 lines

### User Guide
- [ ] Follows template structure
- [ ] Clear step-by-step instructions
- [ ] FAQ answers common questions
- [ ] Troubleshooting section included
- [ ] Under 200 lines

### CLAUDE.md Updates
- [ ] New patterns added to relevant file
- [ ] "Do Not" rules added if mistakes were made
- [ ] Tech stack updated if dependencies changed

---

## Update TASKS.md

Move task to "Approved" section:

```markdown
## Approved

| Task | Task Doc | Feature Doc | Test Report | Approved |
|------|----------|-------------|-------------|----------|
| Quick Actions Redesign | [task](docs/task/...) | [feature](docs/features/...) | [test](docs/testing/...) | Jan 25 |
```

---

## What to Include

- **Current state only** - Document what exists now
- **Accurate file paths** - Verify paths exist
- **Working examples** - Only code that matches production
- **Clear user journeys** - What users actually do

## What to Exclude

| Remove | Why |
|--------|-----|
| Development logs | Move to changelogs |
| Before/after comparisons | Only document current state |
| "Lessons learned" | Dev notes, not docs |
| Speculative future phases | Keep minimal |
| Full component code | Code lives in codebase |

---

## Handoff to /ship

**Check the task document for `Automation: auto` field.**

### Manual Mode
```
Documentation updated for: {task-name}

Updated files:
- docs/features/{feature}.md (created/updated)
- docs/guides/{feature}.md (created/updated)
- CLAUDE.md (if updated)

Task moved to "Approved" in TASKS.md

Ready to ship. Run: /ship {task-name}
```

### Auto Mode
```
Documentation updated for: {task-name}

Updated files:
- docs/features/{feature}.md (created/updated)
- docs/guides/{feature}.md (created/updated)
- CLAUDE.md (if updated)

Task moved to "Approved" in TASKS.md

[AUTO] Spawning /ship with haiku model...
```
Use Task tool: `Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/ship {task-name}" })`

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/implement` | If issues found during doc review |
| `/ship` | After documentation complete |
| `/react-best-practices` | Reference React patterns for docs |
| `/postgres-best-practices` | Reference database patterns for docs |
