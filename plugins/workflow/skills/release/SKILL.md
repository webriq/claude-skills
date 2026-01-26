---
name: release
description: Create versioned releases with consolidated changelogs. Gathers shipped items, generates CHANGELOG.md entry, creates git tag, and GitHub Release. Use after multiple /ship tasks are merged.
---

# /release - Release Management Agent

## When to Use

Invoke `/release` when:
- Multiple features/fixes have been merged via `/ship`
- Items are in "Ready to Ship" section of TASKS.md
- Ready to create a versioned release

**Examples:**
```bash
/release             # Auto-determine version from task docs (RECOMMENDED)
/release auto        # Same as above - auto-determine
/release patch       # Force patch increment (v1.1.21 → v1.1.22)
/release minor       # Force minor increment (v1.1.21 → v1.2.0)
/release major       # Force major increment (v1.1.21 → v2.0.0)
/release v1.1.22     # Explicit version
```

## Workflow

```
/release [version|auto]
       ↓
1. Get current version from git tags
2. Read "Ready to Ship" items from TASKS.md
3. Read each task document for Type & Version Impact
4. Auto-calculate version (if not explicit)
   ├── Any major → major bump
   ├── Any minor → minor bump
   └── All patch → patch bump
5. Categorize changes by type
6. Generate changelog entry
7. Update CHANGELOG.md
8. Commit changelog
9. Create git tag
10. Push to remote
11. Create GitHub Release
12. Move items to "Shipped" in TASKS.md
       ↓
Release complete!
```

## Pre-Release Checklist

Before running `/release`:

- [ ] All PRs in "Ready to Ship" are merged
- [ ] Main branch is up to date (`git pull`)
- [ ] No uncommitted changes (`git status`)
- [ ] All tests passing

## Step-by-Step Process

### Step 1: Get Current Version

```bash
# Get the latest tag
git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0"

# List recent tags for context
git tag --list --sort=-v:refname | head -5
```

### Step 2: Determine New Version

**If explicit version provided:**
```bash
/release v1.1.22
# Use v1.1.22 directly
```

**If semantic increment provided:**
```bash
/release patch  # v1.1.21 → v1.1.22
/release minor  # v1.1.21 → v1.2.0
/release major  # v1.1.21 → v2.0.0
```

**Version parsing logic:**
```
Current: v1.1.21
         │ │ │
         │ │ └── Patch (bug fixes, small changes)
         │ └──── Minor (new features, backwards compatible)
         └────── Major (breaking changes)
```

### Step 3: Auto-Calculate Version (Recommended)

When using `/release` or `/release auto`, read each task document to determine version:

**Read Version Impact from task docs:**
```markdown
> **Type:** feature
> **Version Impact:** minor  ← Use this field
```

**Auto-calculation logic:**
```
Ready to Ship items:
├── feature-1      → Version Impact: minor
├── bug-fix-1      → Version Impact: patch
└── enhancement-1  → Version Impact: patch

Highest impact = minor
Current version = v1.2.0
New version = v1.3.0
```

**Priority order:**
1. `major` (any major = major bump)
2. `minor` (any minor, no major = minor bump)
3. `patch` (all patch = patch bump)

**Fallback if Version Impact missing:**
- Use Type field to infer:
  - `feature` → `minor`
  - `bugfix` → `patch`
  - `enhancement` → `patch`
  - `documentation` → `patch`
  - `chore` → `patch`

### Step 4: Read "Ready to Ship" Items

Parse TASKS.md for items in "Ready to Ship" section:

```markdown
## Ready to Ship

| Task | Branch | PR | Task Doc | Approved |
|------|--------|----| ---------|----------|
| Quick Actions Redesign | feature/quick-actions | #123 | [link](...) | Jan 25 |
| Session Fix | fix/session-persist | #124 | [link](...) | Jan 25 |
```

### Step 4: Categorize Changes

Read each task document to get the `Type` field:

| Type | Changelog Section |
|------|-------------------|
| `feature` | New Features |
| `bugfix` | Bug Fixes |
| `enhancement` | Enhancements |
| `documentation` | Documentation |
| `chore` | Other Changes |

**Task document type field:**
```markdown
> **Status:** SHIPPED
> **Type:** feature
```

### Step 5: Generate Changelog Entry

Create formatted entry:

```markdown
## [v1.1.22] - January 26, 2026

### New Features
- **Web:** Dashboard redesign with new widgets (#123)

### Bug Fixes
- Fixed session persistence on embed chat (#124)

### Enhancements
- Improved booking calendar performance (#125)

### Documentation
- Updated authentication guide (#126)
```

### Step 6: Update CHANGELOG.md

**Location:** `docs/changelogs/CHANGELOG.md`

Insert new version at the top (after header):

```markdown
# Changelog

All notable changes to this project.

## [v1.1.22] - January 26, 2026
← INSERT HERE

### New Features
...

---

## [v1.1.21] - January 20, 2026
...
```

**If CHANGELOG.md doesn't exist, create it:**

```markdown
# Changelog

All notable changes to this project.

Format based on [Keep a Changelog](https://keepachangelog.com/).

---

## [v1.1.22] - January 26, 2026

### New Features
...
```

### Step 7: Commit Changelog

```bash
git add docs/changelogs/CHANGELOG.md
git commit -m "chore(release): v1.1.22

Release v1.1.22 with:
- Quick Actions grid redesign
- Session persistence fix
- [other items...]

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

### Step 8: Create Git Tag

```bash
# Create annotated tag with message
git tag -a v1.1.22 -m "Release v1.1.22

New Features:
- Quick Actions grid redesign (#123)

Bug Fixes:
- Session persistence fix (#124)"
```

### Step 9: Push to Remote

```bash
# Push commit
git push origin main

# Push tag
git push origin v1.1.22
```

### Step 10: Create GitHub Release

```bash
gh release create v1.1.22 \
  --title "v1.1.22" \
  --notes "## What's New

### New Features
- **Web:** Dashboard redesign with new widgets (#123)

### Bug Fixes
- Fixed session persistence on embed chat (#124)

---

**Full Changelog:** https://github.com/owner/repo/compare/v1.1.21...v1.1.22"
```

### Step 11: Update TASKS.md

Move items from "Ready to Ship" to "Shipped":

```markdown
## Ready to Ship

| Task | Branch | PR | Task Doc | Approved |
|------|--------|----| ---------|----------|
| - | - | - | - | - |

---

## Shipped (January 2026)

| Task | PR | Shipped | Release |
|------|-----|---------|---------|
| Quick Actions Grid Redesign | #123 | Jan 26 | v1.1.22 |
| Session Persistence Fix | #124 | Jan 26 | v1.1.22 |
```

---

## Changelog Format Guidelines

### Changelog Entry Structure

```markdown
## [vX.Y.Z] - Month DD, YYYY

### New Features
- **Scope:** Description (#PR)

### Bug Fixes
- Description (#PR)

### Enhancements
- Description (#PR)

### Documentation
- Description (#PR)

### Breaking Changes (if any)
- Description of what breaks and migration path
```

### Writing Good Changelog Entries

**Do:**
- Start with verb (Added, Fixed, Improved, Updated)
- Include scope when helpful (Web, API)
- Reference PR number
- Be user-focused (what changed for them)

**Don't:**
- Include internal refactors users don't see
- Be too technical
- Include duplicate entries

**Examples:**
```markdown
# Good
- **Web:** Added dashboard widgets with custom icons (#123)
- Fixed booking calendar not loading on slow connections (#124)

# Bad
- Updated home.tsx
- Refactored Dashboard component
```

---

## Summary Output

After completing `/release`:

```
Release v1.1.22 created successfully!

Changes included:
- New Features: 2
- Bug Fixes: 1
- Enhancements: 1
- Documentation: 1

Files updated:
- docs/changelogs/CHANGELOG.md
- TASKS.md

Git:
- Tag: v1.1.22
- Commit: abc1234

GitHub:
- Release: https://github.com/owner/repo/releases/tag/v1.1.22

Next release will be: v1.1.23 (patch) / v1.2.0 (minor) / v2.0.0 (major)
```

---

## Handling Edge Cases

### No Items in "Ready to Ship"

```
Error: No items found in "Ready to Ship" section.

Please ensure:
1. Features have been merged via /ship
2. TASKS.md has items in "Ready to Ship" section
```

### Tag Already Exists

```
Error: Tag v1.1.22 already exists.

Options:
1. Use a different version: /release v1.1.23
2. Delete existing tag (if mistake): git tag -d v1.1.22
```

### Missing Task Type

If a task document doesn't have a `Type` field:
- Default to "enhancement"
- Warn user to add types for better categorization

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/ship` | Before /release - gets items to "Ready to Ship" |
| `/plan` | Add `Type` field when planning tasks |
| `/document` | Ensure docs are updated before release |
| `/react-best-practices` | React/Next.js optimization reference |
| `/postgres-best-practices` | Database best practices reference |

---

## Task Document Type Field

When using `/plan`, include the Type field:

```markdown
> **Status:** PLANNED
> **Priority:** HIGH
> **Type:** feature | bugfix | enhancement | documentation | chore
> **Created:** January 25, 2026
> **Platform:** Web
```

This enables automatic categorization in changelogs.
