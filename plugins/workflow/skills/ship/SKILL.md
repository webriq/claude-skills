---
name: ship
description: Deploy approved features to production. Creates PRs, runs pre-deployment checks, and updates TASKS.md. Use after /document completes.
model: haiku
---

# /ship - Deployment Agent

> **Model:** haiku (scripted deployment commands)

## When to Use

Invoke `/ship {task-name}` when:
- Task is in "Approved" status in TASKS.md
- Documentation is complete
- Ready to deploy to production

**Example:** `/ship dashboard-redesign`

## Workflow

```
/ship {task-name}
       ↓
1. Verify task is approved
2. Check Automation field (manual | auto)
3. Move to "Ready to Ship" in TASKS.md
4. Run pre-deployment checks
5. Create/update branch
6. Create Pull Request
7. Update TASKS.md with PR link
       ↓
After merge → Move to "Shipped"
```

## Auto Mode Behavior

When task document has `Automation: auto`:

After PR is created, the automation pipeline completes:
```
[AUTO] Pipeline complete!

Task: {task-name}
Branch: feature/{task-name}
PR: #{number} - {link}

Pre-deployment checks:
- Build: PASS
- TypeScript: PASS
- Lint: PASS

The PR is ready for your review and merge.
TASKS.md updated to "Ready to Ship"
```

**Note:** In auto mode, we still create the PR and notify you - you decide when to merge. The automation does NOT auto-merge.

## Pre-Deployment Checklist

### 1. Verify Approval Status

Check TASKS.md:
- Task must be in "Approved" section
- Feature doc exists
- Test report shows PASS

### 2. Review Changes

```bash
git status
git diff main...HEAD
```

Verify:
- All intended files are changed
- No unintended changes
- No sensitive files (`.env`, credentials)

### 3. Run Pre-Deployment Checks

```bash
# Build check
pnpm build

# Type check
pnpm typecheck

# Lint check
pnpm lint
```

All must pass before creating PR.

---

## Update TASKS.md

Move task to "Ready to Ship":

```markdown
## Ready to Ship

| Task | Branch | PR | Task Doc | Approved |
|------|--------|----| ---------|----------|
| Quick Actions Redesign | feature/quick-actions | Pending | [link](...) | Jan 25 |
```

---

## Git Workflow

### Branch Strategy

Use descriptive branch names:
```
feature/{task-name}     - New features
fix/{task-name}         - Bug fixes
enhance/{task-name}     - Enhancements
```

### If Branch Doesn't Exist

```bash
git checkout -b feature/{task-name}
git add -A
git commit -m "feat: {description}

{Detailed description of changes}

Task: docs/task/{task-name}.md
Test: docs/testing/{task-name}.md

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push -u origin feature/{task-name}
```

### If Branch Exists

```bash
git checkout feature/{task-name}
git add -A
git commit -m "feat: {description}

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push
```

---

## Create Pull Request

Use GitHub CLI to create PR:

```bash
gh pr create --title "{PR Title}" --body "$(cat <<'EOF'
## Summary

{2-3 bullet points describing the changes}

## Task Documentation

- **Task:** [docs/task/{task-name}.md](link)
- **Test Report:** [docs/testing/{task-name}.md](link)
- **Feature Doc:** [docs/features/{feature}.md](link)

## Changes

| File | Change |
|------|--------|
| `path/to/file` | Description |

## Testing

- [x] E2E tests passed (see test report)
- [x] Manual testing completed
- [ ] Ready for code review

## Screenshots

{If UI changes, include before/after}

---

Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

---

## PR Checklist

Before creating PR, verify:

### Code Quality
- [ ] Build passes (`pnpm build`)
- [ ] No TypeScript errors
- [ ] Lint passes (`pnpm lint`)
- [ ] No console.log statements
- [ ] No commented-out code

### Documentation
- [ ] Task document complete
- [ ] Test report shows PASS
- [ ] Feature documentation updated
- [ ] User guide updated (if user-facing)

### Security
- [ ] No hardcoded secrets
- [ ] No `.env` files included
- [ ] API keys not exposed
- [ ] Proper auth checks in place

### Testing
- [ ] E2E tests pass
- [ ] Manual testing done
- [ ] Edge cases handled

---

## Update TASKS.md with PR

After PR is created:

```markdown
## Ready to Ship

| Task | Branch | PR | Task Doc | Approved |
|------|--------|----| ---------|----------|
| Quick Actions Redesign | feature/quick-actions | [#123](link) | [link](...) | Jan 25 |
```

---

## Post-Merge Actions

After PR is merged:

### 1. Update TASKS.md

Move to "Shipped" section:

```markdown
## Shipped (January 2026)

| Task | PR | Shipped | Notes |
|------|-----|---------|-------|
| Dashboard Redesign | [#123](link) | Jan 26 | Homepage update |
```

### 2. Clean Up Task Document

Update task document status:

```markdown
> **Status:** SHIPPED
> **Shipped:** {Date}
> **PR:** #{number}
```

### 3. Archive or Keep

- Keep task doc in `docs/task/` for reference
- Or move to `docs/archive/tasks/` if desired

---

## Handling Issues

### Build Fails

1. Fix the build errors
2. Commit fixes
3. Re-run checks
4. Continue with PR

### PR Review Requested Changes

1. Make requested changes
2. Commit with descriptive message
3. Push to branch
4. Re-request review

### Merge Conflicts

1. Fetch latest main
2. Rebase or merge main into branch
3. Resolve conflicts
4. Push updated branch

---

## Deployment Verification

After merge, verify deployment:

### Vercel (Web)
- Check Vercel dashboard for deployment status
- Verify preview URL works
- Check production URL after deploy

---

## Rollback Procedure

If issues found in production:

1. **Quick fix:** Create hotfix branch, PR, merge
2. **Revert:** `git revert {commit}` and create PR
3. **Document:** Add to test report what was missed

---

## Summary Output

**Check the task document for `Automation: auto` field.**

### Manual Mode
```
Deployment initiated for: {task-name}

Branch: feature/{task-name}
PR: #{number} - {link}

Pre-deployment checks:
- Build: PASS
- TypeScript: PASS
- Lint: PASS

TASKS.md updated to "Ready to Ship"

After merge, I'll update status to "Shipped"
```

### Auto Mode
```
[AUTO] Pipeline complete!

Task: {task-name}
Automation: Full pipeline executed

Summary:
├── /plan ✓ (task document created)
├── /implement ✓ (code written)
├── /test ✓ (tests passed)
├── /document ✓ (docs updated)
└── /ship ✓ (PR created)

Branch: feature/{task-name}
PR: #{number} - {link}

Pre-deployment checks:
- Build: PASS
- TypeScript: PASS
- Lint: PASS

The PR is ready for your review and merge.
```

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/implement` | If fixes needed before shipping |
| `/test` | If additional testing needed |
| `/document` | If docs need updates |
| `/release` | After multiple items merged, create versioned release |
| `/react-best-practices` | For React/Next.js code fixes |
| `/postgres-best-practices` | For database-related fixes |
