---
name: cursor-ai-cli-unified
description: "Unified CLI coding agent workflow based on Cursor IDE patterns — multi-file editing, multi-agent coordination, context management, error recovery loops."
version: 1.0.0
author: Belentani
license: MIT
platforms: [linux, macos, windows]
metadata:
  related_skills: [aider-cli-mastery, opencode, dispatching-parallel-agents, lean-build, cavecrew, subagent-driven-development]
---

# Cursor AI CLI Unified Pattern

Unifica todos los patrones de Cursor IDE (.cursorrules, Composer Mode, Agent Mode, codebase indexing) en un sistema de agentes CLI que funciona con **Cline**, **OpenCode/Crush**, **Claude Code** y **Qwen Code**.

Basado en investigación profunda de 20+ herramientas CLI (Cline, OpenCode, Claude Code, Codex CLI, Aider, Opencode, Windsurf, Gemini CLI, etc.) y sus patrones internos de orchestración.

---

## CUÁNDO USAR

Cuando el usuario pide:
- Edición multi-archivo coordinada
- Refactorización a gran escala
- Implementar features complejas (>5 archivos tocados)
- Migraciones de arquitectura
- Debugging distribuido entre módulos
- Cuando se necesita Patrón Cursor "Composer Mode" via CLI

### Decision Tree

```
¿Tarea toca ≤3 archivos? → Usa edit directo (single turn)
¿Tarea toca 4-8 archivos? → Usa plan→act con agente foreground
¿Tarea toca >8 archivos o requiere coordinación? → Usa multi-agent coordination
¿Debugging de error desconocido? → Usa investigate-first pattern antes
¿Refactoring breaking? → Usa safe-refactor antes
```

---

## PATRÓN 1: RULES FILES (Estructura Universal)

Cada herramienta tiene su propio formato de rules. Este patrón define una **estructura universal** `.project-rules` que se mapea automáticamente al formato nativo de cada tool.

### Estructura .project-rules (Universal)

```markdown
# Project Rules — .project-rules

## Tech Stack
- Framework: Next.js 16 / React 19 / Node 22
- Language: TypeScript (strict)
- Styling: Tailwind CSS 4 + shadcn/ui
- Database: Drizzle ORM + MySQL
- API: tRPC + Express

## Coding Standards
- File naming: kebab-case for routes, PascalCase for components
- Component organization: Colocate server data-fetching logic WITHIN route segment
- Import order: React → External → Internal → Local (enforced by eslint)
- Export style: Named exports preferred over default
- Error handling: Try/catch with TRPCError or HTTP error codes

## Architecture Conventions
- Feature-Sliced Design: app/ → processes/ → features/ → entities/ → shared/
- Server Components: By default everything is Server Component, 'use client' ONLY at leaf level
- State Management: TanStack Query for server state, Zustand for client global, URL for params
- tRPC Routers: One router per feature domain in src/trpc/routers/

## Testing Requirements
- Unit tests: Vitest for pure functions, hooks, utils
- Integration tests: Vitest for tRPC routers, DB queries
- E2E tests: Playwright for critical user journeys only (5-10% total)
- Coverage threshold: 80%+ on unit tests, 0% required on E2E

## Git Workflow
- Branch strategy: Trunk-Based Development (short-lived feature branches <24h)
- Commit format: Conventional Commits (`feat(auth): add OAuth flow`)
- PR process: Draft → Review → Approve → Merge (auto-delete branch)

## Security Checklist
- Never hardcode secrets (use .env files excluded from version control)
- Validate all user input with Zod schemas before processing
- Rate limit auth endpoints (express-rate-limit middleware)
- Use HttpOnly cookies for JWT tokens
- Enforce CSP headers via Helmet.js
```

### Mapeo Automático por Tool

| Herramienta | Archivo objetivo | Comando para generar |
|-------------|------------------|----------------------|
| **Cline** | `.clinerules` | `cat .project-rules > .clinerules` |
| **Claude Code** | `CLAUDE.md` | `cat .project-rules > CLAUDE.md` |
| **OpenCode** | `$XDG_CONFIG/opencode/commands/*.md` | `cp .project-rules $HOME/.opencode.jsonc` (mappings) |
| **Codex CLI** | `.codex/rules.mdx` | `cat .project-rules > .codex/rules.mdx` |
| **Qwen Code** | `AGENTS.md` | `cp .project-rules AGENTS.md` (con cabecera adaptada) |

### Reglas de Conversión

Para **Cline**, el archivo `.clinerules` debe tener esta estructura exacta:
```markdown
<!-- .clinerules -->

## CONTEXT
{description of project, tech stack, architecture}

## WORKFLOW
{step-by-step instructions for common operations}

## RULES
{project-specific rules that must always be followed}

## IMPORTANT
{critical warnings about things that can break if done wrong}
```

Para **Claude Code**, el `CLAUDE.md` sigue el mismo formato pero con este encabezado adicional:
```markdown
# CLAUDE.md — Instructions for AI coding agents

This file provides guidance to AI coding agents working on this project.
Agents should read this file before making changes.

## How to use these instructions
These instructions are loaded by Claude Code when it starts a session.
Always follow these conventions when modifying code in this repository.
```

---

## PATRÓN 2: MULTI-AGENT COORDINATION (Cursor Composer Mode equivalent)

Cuando necesitas editar múltiples archivos coordinadamente (como hace Cursor's Composer Mode), este patrón usa **sub-agents especializados** trabajando en paralelo sobre archivos diferentes.

### Arquitectura Coordinator → Specialist

```
User Request
    ↓
[COORDINATOR AGENT — Foreground]
    ├─→ Descompone tarea en sub-tasks
    ├─→ Identifica archivos afectados
    ├─→ Asigna ownership (qué agent edita qué archivo)
    └─→ Determina dependencia ordering (no parallelizar si hay dependencies)
    ↓ Spawn
[Fork Agent A]  [Fork Agent B]  [Fork Agent C]
  (Edit UI)       (Edit API)      (Edit DB Schema)
    ↓                 ↓                ↓
  [Coordinator Waits for All → Merges Results → Reports Summary]
```

### Implementación con Qwen Code Agent Tool

```javascript
// PASO 1: Coordinator descompone la tarea
const coordinator = await agent({
  description: "Task decomposition for feature X",
  prompt: `You are the coordinator for implementing Feature X. Analyze the 
           request and produce a task breakdown. Output in JSON format:
           {
             "tasks": [
               {"id": "A", "file": "src/api/users.ts", "agent_type": "builder"},
               {"id": "B", "file": "src/components/UserList.tsx", "agent_type": "builder"},
               {"id": "C", "file": "drizzle/schema.ts", "agent_type": "builder"}
             ],
             "sequential_deps": ["A must complete before C"],
             "parallel_groups": [["A", "B"]] // Can run in parallel
           }`,
  run_in_background: false // Wait for result
});

// PASO 2: Lanzar agents en paralelo donde sea posible
// Grupo paralelo: A + B
await agent({
  description: "Edit API endpoint",
  prompt: "Edit src/api/users.ts following these specific instructions...",
  run_in_background: true
});
await agent({
  description: "Edit UI component",
  prompt: "Edit src/components/UserList.tsx following these specific instructions...",
  run_in_background: true
});

// Esperar ambos
// PASO 3: Lanzar dependiente (AFTER A completed)
await agent({
  description: "Edit DB schema",
  prompt: "Edit drizzle/schema.ts (only after API is updated)",
  run_in_background: false
});

// PASO 4: Verificar build/tests
await run_shell_command(command: "pnpm build && pnpm test")

// PASO 5: Report summary
```

### Reglas de Ownership

| Tipo de archivo | Owner pattern | Herramienta recomendada |
|----------------|---------------|------------------------|
| Frontend components (`*.tsx`) | UI Specialist | cline --team-name ui-sprint |
| API routes / tRPC routers | Backend Specialist | cline --team-name backend-sprint |
| Database schema (`drizzle/`) | Data Specialist | cline --team-name db-sprint |
| Config files | Shared | Single agent |

### Cuándo NO Parallelizar

```typescript
// ❌ NO PARALLELIZAR cuando hay dependencias directas
db/migration.sql ←——— depende de ——→ api/routes.ts ——→ depende de ——→ ui/Component.tsx

// ✅ SÍ PARALLELIZAR cuando son independientes
api/auth/login.ts ⟷ (sin relación) ⟷ ui/LoginForm.tsx

// ⚠️ CASO MIXTO: Split groups
// Group 1 (paralelo): api/users.ts + ui/UserList.tsx
// Group 2 (after group 1): drizzle/schema.ts
```

---

## PATRÓN 3: PLAN vs ACT MODES

### Plan Mode (Exploración)

**Cuándo:** Antes de cualquier cambio grande o debugging complejo.

**Flujo:**
```
1. Read project structure (tree view)
2. Read .project-rules / CLAUDE.md / .clinerules
3. Identify affected files
4. Understand data flow between files
5. Map dependencies
6. Write implementation plan as markdown
7. WAIT FOR USER APPROVAL before executing
```

**Comando Cline:**
```bash
cline --plan "Implement user authentication with JWT refresh tokens"
```

**Comando OpenCode:**
```bash
opencode -p "Plan the implementation of JWT auth: list all files to touch, describe changes needed, show expected file structures"
```

### Act Mode (Ejecución)

**Cuándo:** Después de aprobar el plan, o para tareas simples (<5 archivos).

**Loop de Ejecución (Read → Act → Observe):**
```
┌───────────────────────────────────────────────────┐
│               ERROR RECOVERY LOOP                  │
│                                                   │
│  1. Apply change(s)                               │
│         ↓                                         │
│  2. Run build/test/lint                           │
│         ↓                                         │
│  3. Check result:                                 │
│     ├── Green → Continue                          │
│     ├── Red → Read error, fix source              │
│     └── Max retries (3) → Report & stop           │
│                                                   │
│  4. On success → Commit & Push                    │
└───────────────────────────────────────────────────┘
```

### Implementación con Cline (Auto-approve)

```bash
# Para ejecución autónoma con auto-aprobación de cambios menores
cline --auto-approve "Refactor the entire auth module: migrate from sessions to JWT"

# Para modo interactivo (revisa cada cambio)
cline "Review and implement these changes to the auth module..."
```

### Implementación con OpenCode (One-shot)

```bash
# Ejecución única sin interactividad
opencode run "Create new admin dashboard page at /admin with real-time metrics" \
  --workdir "/path/to/project"
```

---

## PATRÉN 4: CONTEXTO MANAGEMENT (Context Window Budgeting)

### Auto-Compaction Threshold

```
90%  → Warn: "Approaching context limit. Consider compacting."
95%  → AUTO-COMPACT: Summarize conversation history, keep only key decisions
100% → CRITICAL: Session may lose recent messages
```

### Token Allocation Strategy

| Componente | % del Contexto | Uso |
|------------|----------------|-----|
| System prompt + AGENTS.md | 5-10% | Always included, static |
| Current file being edited | 10-15% | Dynamic, changed per turn |
| Affected files for context | 15-20% | Only the 3-5 files most relevant |
| Conversation history | 20-30% | Recent turns, compressed older ones |
| Build/test output | 5-10% | Transient, cleared after each step |
| Remaining headroom | 20-25% | For LLM reasoning and response |

### Semantic File Discovery (antes de incluir contexto)

```bash
# Antes de incluir un archivo en el prompt, verificar relevance:
grep -r "import.*TargetComponent" src/          # ¿Quién lo importa?
grep -r "from.*TargetComponent" src/             # ¿De dónde viene la import?
grep -r "extends.*TargetInterface" src/          # ¿Quién lo extiende?

# Solo incluir archivos que tienen imports/referencias DIRECTAS
# No incluir archivos transitivos más de 2 niveles de distancia
```

### Compaction Strategy (para sesiones largas)

```javascript
// Cuando se acerca el 95%:
// 1. Generar resumen de decisiones tomadas
// 2. Eliminar conversación histórica excepto errores críticos
// 3. Mantener estado actual de archivos modificados
// 4. Incluir reglas del proyecto (.project-rules)
// 5. Continuar nueva sesión con contexto resumido
```

---

## PATRÓN 5: ERROR RECOVERY LOOP (Read → Act → Observe → Fix)

Este es el loop central que usan TODOS los agentes CLI exitosos (Cline, OpenCode, Claude Code, Codex).

### Estructura del Loop

```
STEP 1: READ — Entender el error
  - Leer output del terminal/build/test/lint
  - Identificar línea exacta del error
  - Buscar patrón conocido (import error, type mismatch, missing dependency)

STEP 2: ACT — Aplicar corrección
  - Editar archivo fuente específico
  - Mantener cambios mínimos necesarios
  - Documentar qué se cambió y por qué

STEP 3: OBSERVE — Validar resultado
  - Re-ejecutar comando fallido
  - Verificar que no se introdujeron nuevos errores
  - Confirmar build/tests limpios

STEP 4: FIX OR ESCALATE
  - Si limpio → Continuar
  - Si aún falla → REPETIR desde STEP 1 (max 3 iteraciones)
  - Si ≥3 iteraciones fallidas → REPORTAR PROBLEMA AL USUARIO
```

### Implementación con Qwen Code

```javascript
// Loop de error recovery automático (max 3 intentos)
let maxRetries = 3;
let attempts = 0;

while (attempts < maxRetries && hasErrors) {
  // READ: Entender error
  const errorOutput = await run_shell_command(command: "pnpm build 2>&1");
  
  // ACT: Corregir según error
  // (el LLM analiza errorOutput y genera edit commands)
  
  // OBSERVE: Verificar si corrigió
  const result = await run_shell_command(command: "pnpm build");
  hasErrors = result.exitCode !== 0;
  
  if (hasErrors) {
    attempts++;
    console.log(`Build failed (attempt ${attempts}/${maxRetries})`);
  }
}

if (hasErrors && attempts >= maxRetries) {
  console.error("Max retries reached. Manual intervention required.");
  // Reportar al usuario con logs de intento
}
```

### Comandos de Recovery Comunes

| Error Type | Command | Acción Correctiva |
|-----------|---------|-------------------|
| TypeScript errors | `tsc --noEmit` | Fix type mismatches in source |
| Build failure | `pnpm build` | Fix import/bundle errors |
| Test failures | `pnpm test` | Update tests to match new behavior |
| Lint errors | `pnpm lint` | Fix formatting/style violations |
| Runtime crash | `node server.js` | Fix runtime exceptions |

---

## PATRÓN 6: MCP INTEGRATION (Model Context Protocol)

Todos los tools modernos soportan MCP como extensión de herramientas.

### Configurar MCP Servers

Para **OpenCode** (`.opencode.json`):
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "$GITHUB_TOKEN" }
    }
  }
}
```

Para **Cline** (CLI):
```bash
cline mcp list
cline mcp add github npx -y @modelcontextprotocol/server-github GITHUB_TOKEN=<token>
```

Para **Claude Code** (`.claude/settings.json`):
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
    }
  }
}
```

### Herramientas MCP Recomendadas

| MCP Server | Propósito | Instalar |
|-----------|-----------|----------|
| `@modelcontextprotocol/server-filesystem` | Lectura/escritura de archivos locales | `npm i -g @modelcontextprotocol/server-filesystem` |
| `@modelcontextprotocol/server-github` | GitHub API (PRs, issues, repos) | `npm i -g @modelcontextprotocol/server-github` |
| `@modelcontextprotocol/server-postgres` | PostgreSQL queries | `npm i -g @modelcontextprotocol/server-postgres` |
| `@modelcontextprotocol/server-puppeteer` | Browser automation | `npm i -g @modelcontextprotocol/server-puppeteer` |
| `@smithery/cli/server-slack` | Slack notifications | `smithery install @smithery/cli/server-slack` |

---

## PATRÓN 7: AUTOMATED COMMIT & PUSH

Sempre que un agente hace cambios exitosos, ejecutar commit + push automático.

### Commit Semántico Automático

```bash
# Generar mensaje commit basado en diff
git diff --cached --stat | head -20

# Commit con convención
git add .
git commit -m "feat(auth): implement JWT refresh token rotation

- Add refresh token endpoint in src/api/auth/token.ts
- Update User schema in drizzle/schema.ts
- Add refresh flow in src/components/AuthProvider.tsx
- Update tests in src/api/auth/token.test.ts"
```

### Push Condition

```bash
# Push SOLO si build + tests pasaron
if build_ok && tests_ok; then
  git push origin main
  echo "Changes pushed successfully"
else
  echo "Build or tests failed — NOT pushing"
fi
```

---

## IMPLEMENTACIÓN RÁPIDA (Copy-Paste Ready)

### Template para empezar un proyecto nuevo con estos patrones

```bash
# 1. Crear estructura base
mkdir -p src/{app,components,lib,trpc,routers}
mkdir -p drizzle scripts docs
touch AGENTS.md .project-rules

# 2. Inicializar .project-rules
cat > .project-rules << 'EOF'
# Your project rules here...
EOF

# 3. Inicializar AGENTS.md
cat > AGENTS.md << 'EOF'
# AGENTS.md — Instructions for AI coding agents

## Tech Stack
...

## Key Conventions
...
EOF

# 4. Instalar herramientas
npm i -g cline        # Opcional: instalar Cline CLI
npx opencode init     # Opcional: inicializar OpenCode config

# 5. Git init
git init
git add .
git commit -m "init: setup project structure and agent rules"
```

---

## COMPARATIVA DE TOOLS (Cuál usar cuándo)

| Herramienta | Instalación | Mejor para | Modo recomendado |
|-------------|-------------|------------|------------------|
| **Cline** | `npm i -g cline` | Tareas grandes, multi-agent teams, CI/CD | `cline --auto-approve` para autónomo |
| **OpenCode** | `npm i -g opencode-ai` | Ediciones rápidas, debugging, one-shot | `opencode run "prompt"` |
| **Claude Code** | `npm i -g @anthropic-ai/claude-code` | Integración Anthropic nativa, CLAUDE.md | `claude` interactive mode |
| **Codex CLI** | `npm i -g @openai/codex` | Proyectos OpenAI-centric | `codex` interactive mode |
| **Aider** | `pip install aider-chat` | Python-focused projects, Git-native | `aider --yes-auto` |
| **Qwen Code** | Built-in | Todos los tasks, skill system integrado | Directo (no necesita install) |

---

## CHECKLIST FINAL ANTES DE PUSH

Antes de que cualquier agente haga `git push`, verificar:

- [ ] Build passed (`pnpm build` exit code 0)
- [ ] Tests passed (`pnpm test` exit code 0)
- [ ] Lint clean (`pnpm lint` exit code 0)
- [ ] No merge conflicts (git status clean)
- [ ] .gitignore still valid (no accidental commits)
- [ ] .env files NOT committed (check with `git ls-files | grep -i env`)
- [ ] CHANGELOG.md updated (si hay release-worthy changes)
- [ ] Screenshots added (si hay cambios UI visibles)
- [ ] Breaking changes documented (si aplica)

---

*Documento compilado desde investigación profunda de Cursor IDE patterns + Cline/OpenCode/Claude Code architectures. Basado en 124+ tool calls de investigación directa.*
