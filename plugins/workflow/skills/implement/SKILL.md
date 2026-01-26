---
name: implement
description: Implement tasks from docs/task/*.md. Reads the task document, follows implementation steps, and updates status in TASKS.md. Use after /plan has created a task document.
model: opus
---

# /implement - Implementation Agent

> **Model:** opus (complex coding requires advanced reasoning)

## When to Use

Invoke `/implement {task-name}` when:
- A task document exists in `docs/task/`
- Task is in "Planned" status in TASKS.md
- Ready to start coding

**Example:** `/implement dashboard-redesign`

## Workflow

```
/implement {task-name}
       ↓
1. Read docs/task/{task-name}.md
2. Check Automation field (manual | auto)
3. Move task to "## In Progress" in TASKS.md
4. Invoke specialized skills as needed
5. Implement following task document steps
6. Update status to "TESTING" when complete
       ↓
┌─── Automation Mode? ───┐
│                        │
▼ Manual                 ▼ Auto
Notify user              Invoke /test {task-name}
Ready for /test
```

## Auto Mode Behavior

When task document has `Automation: auto`:

After implementation completes, automatically invoke `/test {task-name}`.

## Pre-Implementation Checklist

Before writing ANY code:

### 1. Read the Task Document
```
docs/task/{task-name}.md
```

Understand:
- Requirements (must have vs nice to have)
- Proposed solution
- File changes needed
- Implementation steps

### 2. Invoke Specialized Skills (As Needed)

For React/Next.js code:
```
/react-best-practices
```

For database queries, RLS, schema:
```
/postgres-best-practices
```

### 3. Update TASKS.md

Move task from "Planned" to "In Progress":

```markdown
## In Progress

| Task | Started | Task Doc | Status |
|------|---------|----------|--------|
| Quick Actions Redesign | Jan 25 | [link](docs/task/...) | Implementing |
```

### 4. Verify Dependencies

Check that all prerequisites exist:
- Required API endpoints
- Required types/interfaces
- Required packages installed
- No blocking tasks

---

## Implementation Guidelines

### Follow the Task Document

The task document is your spec. Follow it step by step:

1. **Step 1** → Complete → Verify
2. **Step 2** → Complete → Verify
3. Continue until all steps done

### Invoke Skills as Needed

| Situation | Skill |
|-----------|-------|
| React/Next.js code | `/react-best-practices` |
| Database work | `/postgres-best-practices` |
| Need clarification | Ask user |

### Quality Standards

- No `any` types - use proper TypeScript types
- All hooks before early returns
- AbortController in useEffect with fetches
- Clean, readable code
- Handle loading/error/empty states

### Track Progress

Update the task document as you go:
- Check off completed requirements
- Note any deviations from plan
- Document blockers encountered

---

## Common Implementation Patterns

### Creating New Files

```typescript
// 1. Create the file
// 2. Add to index exports if needed
// 3. Import where used
```

### Modifying Existing Files

```typescript
// 1. Read the file first (always!)
// 2. Understand existing patterns
// 3. Make minimal, focused changes
// 4. Don't refactor unrelated code
```

### Web Development

- Check project's CLAUDE.md for patterns
- Follow existing code patterns
- Use proper authentication
- Use ORM for database queries, not raw SQL

---

## Completion Checklist

Before marking as ready for testing:

### Code Quality
- [ ] Code compiles without errors
- [ ] No TypeScript errors
- [ ] Follows existing patterns
- [ ] No unnecessary complexity

### Functionality
- [ ] All "Must Have" requirements implemented
- [ ] Happy path works
- [ ] Loading states handled
- [ ] Error states handled
- [ ] Empty states handled

### Task Document
- [ ] Requirements checked off
- [ ] Any deviations documented
- [ ] Notes added for tester

---

## Update Status to TESTING

When implementation is complete:

### 1. Update TASKS.md

Move to "Testing" section:

```markdown
## Testing

| Task | Task Doc | Test Report | Status |
|------|----------|-------------|--------|
| Quick Actions Redesign | [link](docs/task/...) | Pending | Ready for test |
```

### 2. Update Task Document

Add completion notes:

```markdown
> **Status:** TESTING
> **Completed:** {Date}
> **Implementation Notes:** {Any important notes for tester}
```

### 3. Inform User / Chain to Next Skill

**Check the task document for `Automation: auto` field.**

#### Manual Mode (or if Automation field is missing)
```
Implementation complete for: {task-name}

Files changed:
- path/to/file1.tsx (created)
- path/to/file2.tsx (modified)

Ready for testing. Run:
/test {task-name}
```

#### Auto Mode
```
Implementation complete for: {task-name}

Files changed:
- path/to/file1.tsx (created)
- path/to/file2.tsx (modified)

[AUTO] Spawning /test with haiku model...
```
Use Task tool to spawn test agent with **model: haiku**:
`Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/test {task-name}" })`

---

## Handling Issues

### Blocked by Missing Dependency

1. Document the blocker in task document
2. Update TASKS.md status to "Blocked"
3. Inform user with specific blocker details
4. Create sub-task if needed via `/plan`

### Requirement Unclear

1. Ask user for clarification
2. Update task document with clarified requirements
3. Continue implementation

### Scope Creep

1. Stick to task document scope
2. Note additional ideas in "Nice to Have" or separate task
3. Don't expand scope without user approval

---

## Related Skills

| Skill | When to Invoke |
|-------|----------------|
| `/react-best-practices` | React/Next.js code |
| `/postgres-best-practices` | Database queries, RLS, schema |
