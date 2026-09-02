# AGENTS.md — Instructions for AI Coding Agents

This file provides guidance to AI coding agents working on this project.
Agents should read this file before making any changes.

## When to Use This File
Every AI agent (Qwen Code, Claude Code, Cline, OpenCode, etc.) MUST load and follow these instructions before making any code edits, deletions, or additions.

---

## Tech Stack

```
Frontend:    Next.js 16.x (App Router) / React 19 / Vite 7
Styling:     Tailwind CSS 4 + shadcn/ui + Framer Motion
Language:    TypeScript (strict mode)
Backend:     Express.js + tRPC (type-safe API)
Database:    MySQL via Drizzle ORM (migrations in drizzle/)
Auth:        OAuth2 + JWT (HttpOnly cookies)
State:       TanStack Query (server state) + Zustand (client global)
Testing:     Vitest (unit/integration) + Playwright (E2E)
Deploy:      Vercel (frontend) + Railway/Fly.io (backend services)
CI/CD:       GitHub Actions → Preview Env → Production
```

---

## Project Structure

```
src/
├── app/                    # Next.js App Router (file-system routing)
│   ├── layout.tsx          # Root layout (html/body tags)
│   ├── page.tsx            # Home route (/)
│   ├── (marketing)/        # Route group — marketing pages
│   ├── (auth)/             # Route group — login/register
│   ├── dashboard/          # Protected routes
│   └── _components/        # Private folder — NOT routable
├── components/             # Shared UI components
│   ├── ui/                 # shadcn/ui primitives (Button, Input, etc.)
│   └── layout/             # Header, Footer, Sidebar wrappers
├── lib/                    # Shared utilities
│   ├── api/                # API clients, fetch helpers
│   ├── auth/               # Auth utilities (JWT, sessions)
│   └── utils.ts            # General purpose helpers (cn(), formatDate())
├── hooks/                  # Custom React hooks (useAuth, useQuery, etc.)
├── stores/                 # Zustand stores (global client state)
├── types/                  # TypeScript type definitions
└── db/                     # Database schema, migrations, seeds

server/
├── _core/                  # Core server setup (env, db connection, middleware)
├── routers/                # tRPC routers (one per feature domain)
│   ├── auth.router.ts
│   ├── users.router.ts
│   └── admin.router.ts
└── index.ts                # Express entry point

drizzle/
├── schema.ts               # All table definitions (TypeScript DSL)
├── relations.ts            # Table relations
├── meta/                   # Migration metadata
└── migrations/             # SQL migration files (version controlled)
```

---

## Key Conventions

### File Organization

- **Colocation:** Co-locate related files together (page + its component + its data fetcher)
- **Private folders:** Prefix non-routable folders with `_` (e.g., `_components`, `_lib`)
- **Route groups:** Use `(group)` naming for layout partitioning without URL impact
- **File naming:** kebab-case for routes (`user-profile.tsx`), PascalCase for components (`UserProfile.tsx`)

### Component Patterns

- Server Components by default (`async function Page()`)
- `'use client'` ONLY at leaf level where interactivity is needed (event handlers, state)
- Pass data down as props through the server-to-client boundary
- Use `async/await` directly in Server Components (no useEffect pattern needed)

### State Management Rules

- **Server State** → TanStack Query (automatic caching, background refetch, optimistic updates)
- **Client Global** → Zustand (simple key-value store, no boilerplate)
- **URL State** → useSearchParams + URL params (shareable, bookmarkable)
- **NEVER** put server data in Zustand/Redux — causes double-fetching and stale cache
- Form state → React Hook Form + Zod validation

### Database Patterns

- Use Drizzle ORM headless TypeScript DSL (not Prisma)
- Write SQL queries via `.from().where()` syntax
- For complex joins, use `.query.findMany({ with: { ... } })` relational API
- Migrations generated via `drizzle-kit generate --name <change>` 
- Seeds via `tsx scripts/seed.ts`
- NEVER drop/rename columns in production migrations (always add-only, never remove)

### Testing Strategy

- Unit tests: Vitest for pure functions, custom hooks, utility functions
- Integration tests: Vitest for tRPC routers, database queries (with test DB)
- E2E tests: Playwright for critical user journeys only (5-10% of total)
- Aim for 80%+ coverage on unit tests; ignore coverage metrics on E2E
- Follow Kent C. Dodds principles: test behavior, not implementation details
- Never chase 100% coverage — it's a quality indicator, not a goal

### Git & Deployment

- Branch strategy: Trunk-Based Development (short-lived feature branches <24h turnaround)
- Commit messages: Conventional Commits format (`feat(auth): add JWT support`)
- PR process: Draft PR → CI checks pass → Review → Approve → Merge → Auto-delete branch
- Deploy flow: Local Dev → Git Push → GitHub Actions CI → Vercel Preview → Vercel Production
- Canary releases configured via `vercel.json` for gradual rollouts

### Security Checklist (OWASP Top 10:2025)

- [ ] Server-side authorization checked on EVERY request (never trust client)
- [ ] Passwords hashed with bcrypt/scrypt/argon2 (cost factor >= 12)
- [ ] Secrets stored in environment variables, NEVER in code
- [ ] CORS restricted to explicit origins (never `*` in production)
- [ ] Helmet.js sets secure HTTP headers (CSP, X-Frame-Options, HSTS)
- [ ] Rate limiting on auth endpoints (`express-rate-limit`)
- [ ] Input validation with Zod schemas on ALL user input
- [ ] Parameterized SQL queries exclusively (Drizzle ORM handles this)
- [ ] XSS prevention: React escapes by default, avoid `dangerouslySetInnerHTML`
- [ ] HTTPS enforced in production

---

## Common Commands

| Command | Purpose |
|---------|---------|
| `pnpm install` | Install dependencies |
| `pnpm dev` | Start development server |
| `pnpm build` | Build for production |
| `pnpm start` | Start production server |
| `pnpm test` | Run all tests |
| `pnpm test:e2e` | Run Playwright E2E tests |
| `pnpm lint` | Check code style |
| `drizzle-kit generate` | Generate migration files |
| `drizzle-kit migrate` | Apply pending migrations |
| `drizzle-kit studio` | Visualize schema |
| `tsx scripts/seed.ts` | Run seed data |

---

## Important Warnings

- Never hardcode API keys or secrets — always use environment variables
- Never drop/rename database columns in production — always add new ones first
- Never commit `.env.local` files — they contain sensitive credentials
- Never merge to main without passing all CI checks (lint + build + test)
- Never assume a component is server-safe — check for `'use client'` directives before editing
