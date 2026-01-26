# Development Workflow - Quick Reference

## Installation (Plugin Marketplace)

```bash
# 1. Add marketplace (one time)
/plugin marketplace add your-username/claude-skills

# 2. Install plugins
/plugin install workflow@your-username/claude-skills
/plugin install react-best-practices@your-username/claude-skills
/plugin install postgres-best-practices@your-username/claude-skills

# 3. Create docs folders in your project
mkdir -p docs/task docs/testing docs/features docs/guides docs/changelogs

# 4. Download TASKS.md template
curl -o TASKS.md https://raw.githubusercontent.com/your-username/claude-skills/main/TASKS.md
```

**Skill names after install:** `/workflow:plan`, `/workflow:implement`, etc.

---

## The Pipeline

```
/plan → /implement → /test → /document → /ship → /release
```

## Commands

| Command | Model | What it does | Output |
|---------|-------|--------------|--------|
| `/plan` | opus | Plan new feature/fix (manual mode) | `docs/task/*.md` |
| `/plan auto` | opus | Plan + full automation after approval | Auto pipeline |
| `/implement {task}` | opus | Code the task | Working feature |
| `/test {task}` | haiku | Run web E2E tests (Playwright) | `docs/testing/*.md` |
| `/document {task}` | haiku | Update docs | `docs/features/*.md` |
| `/ship {task}` | haiku | Create PR | Pull Request |
| `/release` | - | Auto-version release | Tag + Changelog |

## Auto Mode

```bash
/plan auto    # After approval: implement → test → document → ship (automatic)
```

**Full automation** through the pipeline
**Test failures:** Auto-retries with test report context

## Task Status Flow

```
PLANNED → IN_PROGRESS → TESTING → APPROVED → READY_TO_SHIP → SHIPPED
```

## Task Types (for changelog)

| Type | Use for |
|------|---------|
| `feature` | New functionality |
| `bugfix` | Bug fixes |
| `enhancement` | Improvements |
| `documentation` | Doc updates |
| `chore` | Maintenance |

## Version Bumps

```bash
/release          # Auto-determine from task docs (RECOMMENDED)
/release auto     # Same as above
/release patch    # Force v1.0.0 → v1.0.1
/release minor    # Force v1.0.0 → v1.1.0
/release major    # Force v1.0.0 → v2.0.0
```

## Version Impact (set in /plan)

| Type | Default Impact | Bump |
|------|----------------|------|
| `feature` | `minor` | v1.0 → v1.1 |
| `bugfix` | `patch` | v1.0.0 → v1.0.1 |
| `enhancement` | `patch` | v1.0.0 → v1.0.1 |
| Breaking change | `major` | v1.0 → v2.0 |

## Specialized Skills

| Skill | When |
|-------|------|
| `/react-best-practices` | React/Next.js code |
| `/postgres-best-practices` | Database queries, RLS, schema |

## Adding New Skills

```bash
# 1. Create skill folder
mkdir -p {skill-name} && touch {skill-name}/SKILL.md

# 2. Update these files to reference new skill:
#    - plan/SKILL.md (Related Skills section)
#    - implement/SKILL.md (Related Skills section)
#    - README.md (Specialized Skills table)
#    - QUICK_REFERENCE.md (this file)
#    - TASKS.md (Skills Reference)
```

See README.md "Adding New Specialized Skills" for full template.

## Key Files

| What | Where |
|------|-------|
| Task docs | `docs/task/*.md` |
| Test reports | `docs/testing/*.md` |
| Feature docs | `docs/features/*.md` |
| Changelog | `docs/changelogs/CHANGELOG.md` |
| Status tracker | `TASKS.md` |

## Common Patterns

```bash
# New feature (manual - you control each step)
/plan
/clear
/implement my-feature
/test my-feature
/document my-feature
/ship my-feature

# New feature (auto - hands-off after approval)
/plan auto
# Approve the plan → automation handles the rest

# Quick fix
/plan  # (set Type: bugfix, Version Impact: patch)
/implement the-fix
/test the-fix
/ship the-fix

# Quick fix (auto)
/plan auto  # (set Type: bugfix, Version Impact: patch)
# Approve → auto runs through pipeline → PR ready

# Release after multiple ships
/release         # Auto-version from task docs
/release auto    # Same as above
```
