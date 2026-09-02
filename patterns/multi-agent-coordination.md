# Multi-Agent Coordination — Deep Guide

## When to Use Multi-Agent Pattern

Use when:
- Task touches **4+ files** across multiple domains (UI, API, DB)
- Files are **independent** (no sequential dependencies between them)
- Speed is important (parallel work saves time)
- Each file edit can be described independently

Do NOT use when:
- Files have **direct dependencies** (one agent needs another's output)
- Task is **small** (<4 files) — overhead outweighs benefit
- Risk of **merge conflicts** (agents editing same file)

---

## Architecture Patterns

### Pattern 1: Coordinator → Specialist (Recommended)

```
User Request
    ↓
[COORDINATOR — Foreground]
   ├─→ Decomposes task into subtasks
   ├─→ Maps file ownership
   ├─→ Identifies dependency ordering
   └─→ Outputs structured task plan
    ↓ Spawn
[Fork Agent A] ←→ [Fork Agent B] ←→ [Fork Agent C]
  (Edit UI)         (Edit API)        (Edit Schema)
    ↓                 ↓                ↓
  [Coordinator Waits for All → Merges Results → Reports Summary]
```

**Implementation:**
```javascript
// Step 1: Coordinate
const plan = await agent({
  description: "Task decomposition",
  prompt: `Analyze request and break into parallelizable subtasks. 
           Output JSON with id, file_path, instructions, depends_on.`,
  run_in_background: false
});

// Parse plan.output.tasks
// Group by depends_on

// Step 2: Launch parallel agents
await Promise.all(
  parallelGroup.map(task => agent({
    description: task.id,
    prompt: task.instructions,
    run_in_background: true
  }))
);

// Wait for all
// Step 3: Launch dependent group
// ...
```

### Pattern 2: Cascading Dependencies

When tasks HAVE dependencies, process in waves:

```javascript
// Wave 1: Independent tasks
await Promise.all([
  agent({ description: "A", prompt: "...", run_in_background: true }),
  agent({ description: "B", prompt: "...", run_in_background: true })
]);

// After wave 1 completes:
// Wave 2: Dependent on A+B
await agent({ description: "C", prompt: "... (after A&B complete)", run_in_background: false });
```

### Pattern 3: Sequential Single Thread

For truly dependent edits (same file), use single agent sequentially:

```javascript
// WRONG: Multiple agents editing same file simultaneously
// CORRECT: Single agent handles all changes to one file

await edit({ file_path: "src/api/users.ts", old_string: "...", new_string: "..." });
await edit({ file_path: "src/components/UserForm.tsx", old_string: "...", new_string: "..." });
// Sequential but FAST for <5 edits in same session
```

---

## File Ownership Rules

Each agent must know EXACTLY which files it owns. No two agents should touch the same file.

| Agent Type | File Pattern | Example |
|-----------|--------------|---------|
| **UI Specialist** | `client/src/components/**\*.tsx` | LoginButton.tsx, DashboardLayout.tsx |
| **API Specialist** | `server/routers/**/*.ts` | auth.router.ts, users.router.ts |
| **Data Specialist** | `drizzle/schema.ts`, `drizzle/*.sql` | New table definition, migration |
| **Config Specialist** | Config files only | `vercel.json`, `.env.example` |

### Rule: If uncertain, ask coordinator

Before an agent starts editing, it MUST confirm ownership with the coordinator if there's ambiguity about who owns a file.

---

## Merge Conflict Prevention

When multiple agents edit files, conflicts occur when:
1. Two agents change the SAME line(s) in same file
2. One agent deletes a file while another modifies it

### Strategies:

**Strategy 1: Non-overlapping ranges**
Agents edit DIFFERENT parts of same file (e.g., top vs bottom of file). Document this in the task plan.

**Strategy 2: Serial execution for shared files**
If multiple changes go to one file, execute SEQUENTIALLY instead of in parallel.

**Strategy 3: Separate files per concern**
Design architecture so each domain has its own file — prevents overlap entirely.

```bash
# Before committing, check for potential conflicts
git diff --name-only HEAD~1 | wc -l      # How many files changed?
git diff --numstat HEAD~1 | awk '{sum+=$1+$2} END {print sum}'  # Total lines changed?

# If >20% of files touched or >500 lines total → review carefully
```

---

## Result Aggregation Strategy

After all agents complete, coordinator must aggregate results:

```
┌────────────────────────────────────────────┐
│           Result Aggregation Flow           │
│                                            │
│  For each completed agent:                 │
│  ├─→ Read its output                       │
│  ├─→ Check exit code (success/fail)        │
│  ├─→ Note any errors encountered            │
│  └─→ Log summary                           │
│                                            │
│  Then verify:                              │
│  ├─→ Did build pass? (`pnpm build`)        │
│  ├─→ Did tests pass? (`pnpm test`)         │
│  └─→ Did lint clean? (`pnpm lint`)         │
│                                            │
│  Finally report:                            │
│  ├─→ SUCCESS: All agents green → commit    │
│  ├─→ PARTIAL: Some failed → Report details │
│  └─→ FAILURE: Critical error → Stop & roll │
└────────────────────────────────────────────┘
```

### Template for Reporting Summary

```markdown
## Agent Execution Summary

### Completed Successfully (3/3)
- [✓] Frontend changes — `src/components/Dashboard.tsx` + `src/components/Metrics.tsx`
- [✓] API changes — `server/routers/metrics.ts`
- [✓] Database changes — `drizzle/schema.ts` + migration SQL

### Verification
- Build: ✅ Passed (0 errors, 2 warnings)
- Tests: ✅ Passed (47 tests, 0 failures)
- Lint: ✅ Clean (0 issues)

### Changes Made
Added real-time metrics dashboard with WebSocket integration...

### Next Steps
Ready for deployment. Run `pnpm deploy` to push to production.
```

---

## Performance Optimization Tips

### Tip 1: Set reasonable timeout expectations
Large parallel tasks may take 3-5 minutes. Don't cancel prematurely.

### Tip 2: Use one-shot mode for independent agents
```bash
opencode run "Update login button styling" --workdir "/project"
# This is faster than interactive mode for single-file edits
```

### Tip 3: Cache agent outputs
If same pattern repeats (e.g., adding new endpoint), save the successful agent prompt for reuse.

### Tip 4: Limit parallelism to CPU cores
Don't spawn more than `(nproc)` agents simultaneously — diminishing returns after ~8 concurrent agents.
