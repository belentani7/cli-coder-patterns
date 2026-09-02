# Plan vs Act Mode — Deep Guide

## Decision Framework

When to use each mode depends on task complexity, file count, and risk of breaking changes.

### When to PLAN First (Exploratory)

Use Plan mode when:
- Task touches **8+ files** across different domains (UI + API + DB)
- Task is a **major architectural change** (migrate frameworks, refactor auth system)
- Debugging **unknown root cause** (intermittent bugs, performance regressions)
- User says "plan this out first" or asks exploratory questions
- Task has **high blast radius** (changing core infrastructure)

```
Plan Flow:
1. Read project structure → tree view
2. Read .project-rules / CLAUDE.md / .clinerules
3. Identify ALL affected files
4. Map data flow between files (where does this data come from?)
5. Check for circular dependencies
6. Write markdown implementation plan
7. Wait for USER approval before executing any code changes
```

### When to ACT Directly (Execution)

Use Act mode when:
- Task touches **≤3 files** in same domain
- Task is **small fix** (typos, minor bug, lint errors)
- Task is **single feature** that fits in one module
- User says "just do it" or "implement X"
- Change is **low risk** (adding new feature without modifying existing code)

```
Act Flow:
1. Apply change(s) directly
2. Run build/test/lint immediately
3. If green → commit + push
4. If red → read error → fix source → re-test (max 3 retries)
5. Report summary
```

### Decision Matrix

| Files Affected | Domain Spread | Risk Level | Mode |
|----------------|---------------|------------|------|
| 1-3 | Single domain | Low | Act |
| 1-3 | Multiple domains | Medium | Plan → Act |
| 4-8 | Single domain | Medium | Plan → Act |
| 4-8 | Multiple domains | High | Plan → Multi-Agent Act |
| 9+ | Any spread | High | Plan → Multi-Agent Act |
| Unknown | N/A | Unknown | Plan first! |

---

## Implementation Examples

### Example 1: Simple Fix → ACT MODE

```javascript
// User: "Add missing alt text to all images"
// Action: Single file edit, no dependencies

await edit({
  file_path: "/src/components/ImageGallery.tsx",
  old_string: "<img src={url} />",
  new_string: "<img src={url} alt={title}" />
});
// No planning needed — single concern, single file
```

### Example 2: Feature Addition → PLAN THEN ACT

```javascript
// User: "Add email verification to the registration flow"
// This affects: database schema, API endpoint, UI component, email template

const plan = await agent({
  description: "Plan email verification feature",
  prompt: `Map all files needed for email verification:
           1. Drizzle schema update (add verifiedAt field)
           2. API endpoint (/api/auth/verify-email)
           3. UI component (email input form)
           4. Email template (verification email HTML)
           Show dependencies between these components.`,
  run_in_background: false
});

// User approves plan → Execute via agents
```

### Example 3: Large Refactor → MULTI-AGENT COORDINATION

```javascript
// User: "Migrate entire app from sessions to JWT authentication"

// Coordinator identifies 12 files across 4 domains
const coordinator = await agent({
  description: "Decompose JWT migration",
  prompt: `Break down JWT migration into subtasks. Output JSON with tasks, 
           parallel groups, and sequential dependencies. Consider:
           - Database schema changes (drizzle/)
           - Auth middleware updates (server/_core/)
           - tRPC router changes (server/routers/)
           - Frontend components (client/src/)
           - Test updates (all *.test.ts files)`,
  run_in_background: false
});

// Execute in dependency order:
// Phase 1 (parallel): Schema + Middleware setup
// Phase 2 (after phase 1): tRPC routers + API endpoints
// Phase 3 (after phase 2): Frontend components
// Phase 4 (after phase 3): Tests + Cleanup
```

---

## Anti-Patterns to Avoid

❌ **Don't Plan simple changes** — Wastes tokens on unnecessary exploration
✅ Instead: Just act on low-risk, small-scope changes

❌ **Don't Act blindly on complex tasks** — Risks introducing bugs
✅ Instead: Always explore scope before changing >3 files

❌ **Don't forget the loop** — After each change, always test
✅ Instead: Use Read → Act → Observe → Fix pattern consistently

❌ **Don't overwrite user's plan** — If user provides their own plan, follow it
✅ Instead: Only generate plan if user hasn't provided one
