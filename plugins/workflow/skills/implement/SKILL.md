---
name: implement
description: Implement tasks from docs/task/*.md. Reads the task document, follows implementation steps, and updates status in TASKS.md. Use "/implement auto {task}" to auto-chain through test → document → ship.
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

## Invocation Options

| Command | Mode | Behavior |
|---------|------|----------|
| `/implement {task-name}` | Manual | Implement, then notify user to run `/test` |
| `/implement auto {task-name}` | Automated | Implement, then auto-chain through test → document → ship |

### Auto Mode

When invoked with `auto`, the implement skill:
1. Sets `Automation: auto` in the task document (overrides any existing value)
2. Implements the task as normal
3. After completion, automatically spawns `/test {task-name}` (haiku)
4. The pipeline continues: test → document → ship

This lets you skip `/task auto` and trigger the full pipeline from implement:
```
/implement auto {task-name} → implement code
    ↓
/test (haiku)
    │
PASS → /document → /ship → PR + notify
FAIL → /implement (with test report) → retry
```

## Workflow

```
/implement [auto] {task-name}
       ↓
1. Parse arguments: detect "auto" flag
2. Read docs/task/{task-name}.md
3. If "auto" flag → set Automation: auto in task doc
4. Check Automation field (manual | auto)
5. Move task to "## In Progress" in TASKS.md
6. ⚠️ MANDATORY: Invoke specialized skills (see Step 2 below)
   └── /vercel-react-best-practices (if installed + React code)
   └── /supabase-postgres-best-practices (if installed + DB code)
7. Implement following task document steps
8. Update status to "TESTING" when complete
       ↓
┌─── Automation Mode? ───┐
│                        │
▼ Manual                 ▼ Auto
Notify user              Invoke /test {task-name}
Ready for /test
```

**⚠️ GUARDRAIL:** Step 6 is NOT optional. If specialized skills are installed and relevant to the task, they MUST be invoked BEFORE writing any code. See "Step 2: Invoke Specialized Skills" below for details.

## Auto Mode Behavior

When invoked with `/implement auto {task-name}` OR task document has `Automation: auto`:

1. If invoked with `auto` argument, update the task document to set `Automation: auto`
2. After implementation completes, automatically invoke `/test {task-name}`

## Pre-Implementation Checklist

Before writing ANY code:

### 0. Parse Arguments

Check if `auto` was passed in the invocation:
- `/implement auto {task-name}` → Set `Automation: auto` in `docs/task/{task-name}.md`
- `/implement {task-name}` → Leave automation field as-is (respect existing value)

If `auto` is detected, update the task document header:
```markdown
> **Automation:** auto
```

### 1. Read the Task Document (Primary Context Source)
```
docs/task/{task-name}.md
```

**IMPORTANT — Context Efficiency:**
The task document was created by the `/task` agent, which already performed a thorough codebase analysis. The task document contains all the context you need: requirements, file paths, implementation steps, and architectural decisions.

- **DO** trust the task document as your primary source of truth
- **DO** read the specific files listed in the task document's "Files to Modify" or implementation steps
- **DO NOT** perform broad codebase exploration (scanning directories, reading unrelated files, trying to "understand the entire codebase")
- **DO NOT** re-analyze architecture that the plan agent already documented

If the task document references specific files, read only those files. This keeps your context window efficient and avoids redundant exploration.

Understand:
- Requirements (must have vs nice to have)
- Proposed solution
- File changes needed
- Implementation steps

### 2. Invoke Specialized Skills (MANDATORY - DO NOT SKIP)

**CRITICAL GUARDRAIL:** Before writing ANY code, you MUST check for and invoke specialized skills. This is NOT optional.

#### Step 2a: Detect Installed Skills

Check if these skills are available by looking for them in the available skills/tools:

| Skill | Detection | Invoke When |
|-------|-----------|-------------|
| `/vercel-react-best-practices` | Skill is listed in available tools | ANY React/Next.js/TypeScript code |
| `/supabase-postgres-best-practices` | Skill is listed in available tools | ANY database queries, RLS, schema, Supabase code |

#### Step 2b: Invoke BEFORE Writing Code

**If skill is detected → MUST invoke it FIRST, before writing any code.**

```
# For React/Next.js projects:
/vercel-react-best-practices

# For Supabase/PostgreSQL projects:
/supabase-postgres-best-practices
```

#### Step 2c: Verification Checklist

Before proceeding to Step 3, confirm:

- [ ] Checked if `/vercel-react-best-practices` is available
- [ ] If available AND task involves React/Next.js → Invoked it
- [ ] Checked if `/supabase-postgres-best-practices` is available
- [ ] If available AND task involves database → Invoked it

#### HARD STOP - Do NOT Proceed If:

```
⛔ STOP: You detected a specialized skill is installed but did not invoke it.

This is a guardrail violation. You MUST:
1. Invoke the skill NOW before writing any code
2. Apply the best practices from the skill to your implementation
3. Only then proceed to Step 3

Skipping specialized skills leads to:
- Suboptimal code patterns
- Performance issues
- Security vulnerabilities
- Technical debt
```

> **Why this matters:** These skills contain critical best practices from Vercel and Supabase engineers. Skipping them means writing code that may have performance issues, security vulnerabilities, or anti-patterns that will need to be fixed later.

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

### Invoke Skills When Applicable (MUST Use If Installed)

| Situation | Skill (Required If Installed) |
|-----------|-------------------------------|
| React/Next.js code | `/vercel-react-best-practices` |
| Database work | `/supabase-postgres-best-practices` |
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

### 3. Pre-Completion Verification (REQUIRED)

**Before marking implementation complete, verify:**

```
┌─────────────────────────────────────────────────────────────┐
│ ⚠️ SPECIALIZED SKILLS VERIFICATION                          │
├─────────────────────────────────────────────────────────────┤
│ □ Did task involve React/Next.js code?                      │
│   → If YES: Did you invoke /vercel-react-best-practices?    │
│                                                             │
│ □ Did task involve database/Supabase code?                  │
│   → If YES: Did you invoke /supabase-postgres-best-practices│
│                                                             │
│ □ If skill was available but NOT invoked:                   │
│   → STOP. Go back and invoke it. Apply best practices.      │
│   → Then return here to complete.                           │
└─────────────────────────────────────────────────────────────┘
```

**If you skipped a specialized skill:** You must go back, invoke it, review your code against the best practices, and make any necessary corrections before proceeding.

### 4. Inform User / Chain to Next Skill

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
4. Create sub-task if needed via `/task`

### Requirement Unclear

1. Ask user for clarification
2. Update task document with clarified requirements
3. Continue implementation

### Scope Creep

1. Stick to task document scope
2. Note additional ideas in "Nice to Have" or separate task
3. Don't expand scope without user approval

---

## Specialized Skills (MANDATORY When Installed)

These plugins must be installed separately. **Once installed, invocation is MANDATORY — not optional.**

| Plugin | Install From | When to Invoke |
|--------|--------------|----------------|
| `vercel-react-best-practices` | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) | ANY React/Next.js/TypeScript code |
| `supabase-postgres-best-practices` | [supabase/agent-skills](https://github.com/supabase/agent-skills) | ANY database queries, RLS, schema, Supabase code |

### Enforcement Summary

```
┌────────────────────────────────────────────────────────────────┐
│ SPECIALIZED SKILLS ARE NOT OPTIONAL ONCE INSTALLED             │
├────────────────────────────────────────────────────────────────┤
│ 1. BEFORE writing code → Check if skills are available         │
│ 2. IF available AND relevant → INVOKE immediately              │
│ 3. APPLY best practices to your implementation                 │
│ 4. BEFORE completing → Verify you invoked required skills      │
│ 5. IF skipped → Go back, invoke, review code, fix issues       │
└────────────────────────────────────────────────────────────────┘
```

Failure to invoke specialized skills when available leads to:
- Code that violates best practices
- Performance issues that need fixing later
- Security vulnerabilities
- Technical debt and rework
