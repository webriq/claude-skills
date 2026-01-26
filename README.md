# Claude Code Development Workflow Skills

> A structured skill-based development pipeline for consistent, high-quality feature delivery.

## Installation (Plugin Marketplace)

### Option 1: Install via Plugin Marketplace (Recommended)

Add this marketplace to Claude Code, then install the plugins you need:

```bash
# Step 1: Add the marketplace (one time)
/plugin marketplace add your-username/claude-skills

# Step 2: Install plugins you want
/plugin install workflow@your-username/claude-skills
/plugin install react-best-practices@your-username/claude-skills
/plugin install postgres-best-practices@your-username/claude-skills
```

### Option 2: Install All Plugins at Once

```bash
# Add marketplace and install everything
/plugin marketplace add your-username/claude-skills
/plugin install workflow react-best-practices postgres-best-practices@your-username/claude-skills
```

### Available Plugins

| Plugin | Skills Included | Description |
|--------|-----------------|-------------|
| `workflow` | `/plan`, `/implement`, `/test`, `/document`, `/ship`, `/release` | Complete development pipeline |
| `react-best-practices` | `/react-best-practices` | React/Next.js optimization patterns |
| `postgres-best-practices` | `/postgres-best-practices` | PostgreSQL/Supabase best practices |

### Skill Names After Installation

Once installed via marketplace, skills are namespaced:

```bash
# Workflow skills
/workflow:plan
/workflow:implement
/workflow:test
/workflow:document
/workflow:ship
/workflow:release

# Best practices skills
/react-best-practices:react-best-practices
/postgres-best-practices:postgres-best-practices
```

### Project Setup (Required)

After installing plugins, create the required folders in your project:

```bash
# Create required docs folders in your project root
mkdir -p docs/task docs/testing docs/features docs/guides docs/changelogs

# Download TASKS.md template
curl -o TASKS.md https://raw.githubusercontent.com/your-username/claude-skills/main/TASKS.md
```

**Folder purposes:**
| Folder | Purpose |
|--------|---------|
| `docs/task/` | Task documents created by `/plan` |
| `docs/testing/` | Test reports created by `/test` |
| `docs/features/` | Feature documentation created by `/document` |
| `docs/guides/` | User guides created by `/document` |
| `docs/changelogs/` | Changelog created by `/release` |
| `TASKS.md` | Task tracker (project root) |

---

## Alternative: Manual Installation

If you prefer not to use the marketplace, copy skills directly:

```bash
# Clone and copy to .claude/skills/
git clone https://github.com/your-username/claude-skills.git
cp -r claude-skills/plugins/workflow/skills/* ~/.claude/skills/
cp -r claude-skills/plugins/react-best-practices/skills/* ~/.claude/skills/
cp -r claude-skills/plugins/postgres-best-practices/skills/* ~/.claude/skills/
```

With manual installation, skills use short names: `/plan`, `/implement`, etc.

## Quick Start

```bash
# Manual Mode - You control each step
/plan
/implement {task-name}
/test {task-name}
/document {task-name}
/ship {task-name}

# Auto Mode - Full automation after plan approval
/plan auto
# → After you approve, runs: implement → test → document → ship automatically
```

---

## Workflow Overview

### Manual Mode (`/plan`)

```
┌────────┐   ┌────────────┐   ┌────────┐   ┌──────────┐   ┌────────┐   ┌──────────┐
│ /plan  │ → │ /implement │ → │ /test  │ → │/document │ → │ /ship  │ → │ /release │
└────────┘   └────────────┘   └────────┘   └──────────┘   └────────┘   └──────────┘
    │              │              │              │              │              │
    ↓              ↓              ↓              ↓              ↓              ↓
docs/task/    In Progress     Testing       Approved      Ready to      Shipped
  *.md                                                      Ship        + Changelog
```

**Manual Checkpoints:**
- After `/test` → **You approve** before `/document`
- After `/document` → **You decide** when to `/ship`
- After `/ship` → **You decide** when to `/release`

### Auto Mode (`/plan auto`)

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
/ship → PR created
```

**Auto Mode Features:**
- **Full automation:** Runs through the entire pipeline automatically
- **Test failures:** Auto-retries by sending test report back to implement
- **PR creation:** Creates PR and notifies you - you decide when to merge

### Model Configuration

Each skill uses an optimized model for its task complexity:

| Skill | Model | Reason |
|-------|-------|--------|
| `/plan` | **opus** | Complex planning requires advanced reasoning |
| `/implement` | **opus** | Complex coding requires advanced reasoning |
| `/test` | **haiku** | Straightforward test execution |
| `/document` | **haiku** | Templated documentation work |
| `/ship` | **haiku** | Scripted deployment commands |

This applies to both manual and auto mode. In auto mode, skills spawn the next agent with the appropriate model using the Task tool.

---

## Skills Reference

### 1. `/plan` - Task Planning

**Purpose:** Discuss requirements and create detailed task documents.

**When to use:**
- Starting a new feature or enhancement
- Planning a bug fix
- User says "I want to add...", "Let's implement..."

**Output:**
- Task document in `docs/task/{task-name}.md`
- Entry in TASKS.md "Planned" section

**Example:**
```
User: I want to add dark mode to the app
Claude: /plan
→ Discusses requirements
→ Creates docs/task/dark-mode.md
→ Adds to TASKS.md
```

---

### 2. `/implement` - Implementation

**Purpose:** Implement tasks following the task document.

**When to use:**
- Task exists in "Planned" section
- Ready to start coding

**Input:** Task name from TASKS.md

**Output:**
- Working implementation
- Task moved to "Testing" section

**Example:**
```
/implement dark-mode
→ Reads task document
→ Implements step by step
→ Updates status to "Testing"
```

---

### 3. `/test` - Web E2E Testing

**Purpose:** Test **web** implementations using Playwright MCP.

**When to use:**
- Web implementation is complete
- Task is in "Testing" status

**Output:**
- Test report in `docs/testing/{task-name}.md`
- PASS → Ready for `/document`
- FAIL → Returns to `/implement`

**Example:**
```
/test user-dashboard
→ Runs E2E tests via Playwright
→ Creates test report
→ Reports PASS/FAIL
```

---

### 4. `/document` - Documentation

**Purpose:** Update project documentation after approval.

**When to use:**
- Tests passed
- User approved the implementation

**Output:**
- Feature doc in `docs/features/`
- User guide in `docs/guides/`
- Updated CLAUDE.md (if needed)

**Example:**
```
/document dark-mode
→ Creates/updates feature doc
→ Creates/updates user guide
→ Moves to "Approved" section
```

---

### 5. `/ship` - Deployment

**Purpose:** Create PRs and prepare for deployment.

**When to use:**
- Documentation complete
- Ready to deploy

**Output:**
- Git branch (if needed)
- Pull Request
- Task in "Ready to Ship" section

**Example:**
```
/ship dark-mode
→ Runs pre-deployment checks
→ Creates PR
→ Updates TASKS.md
```

---

### 6. `/release` - Versioned Releases

**Purpose:** Create versioned releases with consolidated changelogs.

**When to use:**
- Multiple features shipped
- Ready to create a version release

**Output:**
- CHANGELOG.md entry
- Git tag
- GitHub Release
- Items moved to "Shipped"

**Example:**
```
/release v1.2.0
→ Gathers shipped items
→ Categorizes by type
→ Creates changelog entry
→ Creates git tag + GitHub release
```

---

## Task Document Template

When `/plan` creates a task, it uses this structure:

```markdown
# {Task Title}

> **Status:** PLANNED | TESTING | APPROVED | SHIPPED
> **Priority:** HIGH | MEDIUM | LOW
> **Type:** feature | bugfix | enhancement | documentation | chore
> **Created:** {Date}
> **Platform:** Web

## Overview
{2-3 sentence description}

## Requirements
### Must Have
- [ ] Requirement 1
- [ ] Requirement 2

### Nice to Have
- [ ] Optional requirement

## Current State
{Description of how things work now}

## Proposed Solution
{Description of the implementation approach}

### File Changes
| Action | File | Description |
|--------|------|-------------|
| CREATE | `path/to/new.tsx` | New component |
| MODIFY | `path/to/existing.tsx` | Add functionality |

## Implementation Steps
### Step 1: {Title}
{Detailed instructions}

### Step 2: {Title}
{Detailed instructions}

## Testing Checklist
- [ ] Test case 1
- [ ] Test case 2

## Notes for Implementation Agent
{Important context}
```

---

## TASKS.md Structure

The workflow uses these sections in TASKS.md:

```markdown
## Planned
Tasks ready for `/implement`. Created via `/plan`.
| Task | Priority | Task Doc | Created |

## In Progress
| Task | Started | Task Doc | Status |

## Testing
Tasks being tested via `/test`. Returns to `/implement` if FAIL.
| Task | Task Doc | Test Report | Status |

## Approved
Tested and approved. Ready for `/document` then `/ship`.
| Task | Task Doc | Feature Doc | Test Report | Approved |

## Ready to Ship
PRs created via `/ship`. Awaiting merge.
| Task | Branch | PR | Task Doc | Approved |

## Shipped (Month Year)
| Task | PR | Shipped | Release |
```

---

## Specialized Skills

These skills provide domain-specific best practices and can be invoked during `/plan` and `/implement`:

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `/react-best-practices` | React/Next.js performance optimization | React/Next.js code work |
| `/postgres-best-practices` | Database queries, RLS, schema design | Database work, Supabase |

---

## Adding New Specialized Skills

### Skill Types

| Type | Purpose | Examples |
|------|---------|----------|
| **Workflow** | Pipeline stages | `/plan`, `/implement`, `/test`, `/document`, `/ship`, `/release` |
| **Specialized** | Domain best practices | `/react-best-practices`, `/postgres-best-practices` |

**Specialized skills** are invoked by workflow skills when relevant. To add a new one:

### Step 1: Create the Skill Folder

```bash
mkdir -p {skill-name}
touch {skill-name}/SKILL.md
```

### Step 2: Write the SKILL.md

Use this template for best-practices skills:

```markdown
---
name: {skill-name}
description: {Technology} best practices for AI agents. Use when {trigger conditions}.
---

# /{skill-name} - {Technology} Best Practices

## When to Use

Invoke `/{skill-name}` when:
- Writing or modifying {technology} code
- Reviewing {technology} patterns
- Optimizing {technology} performance

## Priority Categories

| Priority | Category | Impact |
|----------|----------|--------|
| 1 | {Category} | CRITICAL |
| 2 | {Category} | HIGH |
| 3 | {Category} | MEDIUM |

---

## 1. {Category Name} (CRITICAL)

### 1.1 {Rule Name}

**BAD:**
```{language}
// Anti-pattern example
```

**GOOD:**
```{language}
// Best practice example
```

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/plan` | Planning {technology} features |
| `/implement` | Implementing {technology} code |
```

### Step 3: Update Workflow Skills

Add the new skill to the "Related Skills" section in these files:

**Files to update:**
- `plan/SKILL.md` - Line ~280 (Related Skills table)
- `implement/SKILL.md` - Line ~260 (Related Skills table)

**Add this row to each table:**
```markdown
| `/{skill-name}` | {When to use description} |
```

### Step 4: Update Documentation

**Update these files:**

1. **README.md** - Add to the Specialized Skills table (this file)
2. **QUICK_REFERENCE.md** - Add to the Specialized Skills section
3. **TASKS.md** - Add to the Skills Reference section

### Example: Adding `/tailwind-best-practices`

```bash
# 1. Create folder
mkdir -p tailwind-best-practices

# 2. Create SKILL.md with best practices content
# (see template above)

# 3. Update plan/SKILL.md Related Skills:
| `/tailwind-best-practices` | Tailwind CSS patterns |

# 4. Update implement/SKILL.md Related Skills:
| `/tailwind-best-practices` | Tailwind CSS code |

# 5. Update README.md Specialized Skills table:
| `/tailwind-best-practices` | Tailwind CSS utility patterns | Styling with Tailwind |

# 6. Update QUICK_REFERENCE.md:
| `/tailwind-best-practices` | Tailwind CSS styling |

# 7. Update TASKS.md Skills Reference:
| `/tailwind-best-practices` | Tailwind CSS patterns |
```

### Skill Ideas

Common specialized skills you might want to add:

| Skill | Purpose |
|-------|---------|
| `/tailwind-best-practices` | Tailwind CSS utility patterns |
| `/typescript-best-practices` | TypeScript type safety patterns |
| `/testing-best-practices` | Unit/integration test patterns |
| `/api-best-practices` | REST/GraphQL API design |
| `/security-best-practices` | Security patterns and validation |
| `/accessibility-best-practices` | WCAG accessibility patterns |

---

## Best Practices

### For Planning (`/plan`)
- Be specific about requirements
- Include acceptance criteria
- Note any dependencies or blockers
- Add the `Type` field for changelog categorization

### For Implementation (`/implement`)
- Follow the task document step by step
- Check off requirements as you complete them
- Note any deviations from the plan
- Don't expand scope without approval

### For Testing (`/test`)
- Test happy path first
- Include edge cases
- Document any issues found
- Be specific in test reports

### For Documentation (`/document`)
- Follow the templates in `docs/templates/`
- Verify file paths exist
- Include both developer and user perspectives
- Keep it concise (feature: 200-400 lines, guide: 100-200 lines)

### For Shipping (`/ship`)
- Run all checks before creating PR
- Include task doc and test report links in PR
- Don't force push to main

### For Releases (`/release`)
- Batch related changes together
- Use semantic versioning appropriately
- Write user-focused changelog entries

---

## Troubleshooting

### "Task not found"
- Check TASKS.md for the task entry
- Verify task document exists in `docs/task/`

### "Tests failing"
- Review the test report in `docs/testing/`
- Return to `/implement` with the test report context

### "Build fails during /ship"
- Fix the issues locally
- Re-run `/ship` after fixes

### "Wrong section in TASKS.md"
- Manually move the task to the correct section
- Or ask Claude to update the status

---

## File Locations

| Content | Location |
|---------|----------|
| Task documents | `docs/task/*.md` |
| Test reports | `docs/testing/*.md` |
| Feature docs | `docs/features/*.md` |
| User guides | `docs/guides/*.md` |
| Changelog | `docs/changelogs/CHANGELOG.md` |
| Workflow tracker | `TASKS.md` |
| Skills | `.claude/skills/*/SKILL.md` |

---

## Questions?

- Check the individual skill files in `.claude/skills/*/SKILL.md`
- Review `CLAUDE.md` for project-specific patterns
- Check `TASKS.md` for current task status
