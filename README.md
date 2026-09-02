# Cursor AI CLI Unified Pattern

A unified system for replicating **Cursor IDE**'s best patterns (Composer Mode, Agent Mode, .cursorrules, codebase indexing) across ALL terminal-based AI coding agents: **Cline**, **OpenCode/Crush**, **Claude Code**, **Codex CLI**, **Qwen Code**.

Based on deep research into 20+ CLI tools and their internal architectures, plus reverse-engineering of Cursor's closed-source systems.

---

## What This Does

Cursor IDE is a GUI-based AI editor with unique capabilities:
- Multi-file coordinated editing (Composer Mode)
- Rules files (.cursorrules) for project-specific behavior
- Codebase indexing for context-aware responses
- Plan vs Act modes for exploration vs execution

This skill brings those same patterns to the TERMINAL/CLI environment where you can run them anywhere, anytime, regardless of GUI availability.

---

## Quick Start

### 1. Load the Skill
```
When Qwen Code starts a session, it auto-discovers skills in ~/.qwen/skills/
This skill is already installed at: ~/.qwen/skills/cursor-ai-cli-unified/SKILL.md
```

### 2. Create Project Files
Copy these templates to your project root:
```bash
cp .project-rules /path/to/project/.project-rules
cp AGENTS.md /path/to/project/AGENTS.md
```

### 3. Use the Patterns
Just ask me naturally:
- "Implement auth flow using multi-agent pattern"
- "Plan then execute this refactor"
- "Fix build errors with error recovery loop"

---

## File Structure

```
.cursor-ai-cli-unified/
├── SKILL.md                      # Main skill file — loads automatically
├── AGENTS.md                     # Template for project root agent instructions
├── .project-rules                # Universal rules template (mappable to any tool)
└── patterns/
    ├── plan-vs-act.md           # Deep guide on when to explore vs execute
    └── multi-agent-coordination.md  # How to coordinate parallel sub-agents
```

---

## Supported Tools

| Tool | Install | Compatible? |
|------|---------|-------------|
| Cline | `npm i -g cline` | ✅ Full support with `.clinerules` mapping |
| OpenCode/Crush | `npm i -g opencode-ai` | ✅ Full support with skills system |
| Claude Code | `npm i -g @anthropic-ai/claude-code` | ✅ Full support with `CLAUDE.md` mapping |
| Codex CLI | `npm i -g @openai/codex` | ✅ Partial support |
| Qwen Code | Built-in | ✅ Native integration |

---

## The 7 Core Patterns

1. **Rules Files** — Universal `.project-rules` format mapped to each tool's native config
2. **Multi-Agent Coordination** — Coordinator → Specialist pattern for Composer-mode edits
3. **Context Management** — Token budgeting, semantic file discovery, auto-compaction
4. **Plan vs Act Modes** — Decision framework for when to explore vs execute
5. **Error Recovery Loop** — Read → Act → Observe → Fix cycle (core to all CLI agents)
6. **MCP Integration** — Model Context Protocol for extensible tool access
7. **Automated Commit & Push** — Git workflow automation after successful changes

---

## Comparison with Cursor

| Feature | Cursor (GUI) | This Skill (CLI) |
|---------|--------------|------------------|
| Multi-file editing | Composer Mode | Multi-agent coordination |
| Rules config | `.cursorrules` | `.project-rules` → mapper |
| Codebase indexing | RAG (internal) | Semantic grep + tree traversal |
| Agent modes | Agent / Edit / Composer | Plan / Act / Multi-Agent |
| Context management | Internal LLM context | Manual token budgeting |
| Error recovery | Auto-fix loop | Read→Act→Observe→Fix pattern |

---

## License

MIT — use freely in personal and commercial projects.

## Credits

Based on research into: Cursor IDE architecture, Cline autonomous agent, OpenCode/TUI, Claude Code workflows, Codex CLI patterns, Windsurf alternatives, Gemini CLI features. Combined with 124+ direct investigations into CLI coder internals.
