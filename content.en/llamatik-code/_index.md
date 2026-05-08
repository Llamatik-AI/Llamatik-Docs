---
title: "Llamatik Code"
weight: 50
bookCollapseSection: true
---

# Llamatik Code

**Llamatik Code** is a local-first AI coding assistant for IntelliJ-based IDEs. It runs models on your machine — your code never leaves your device — and provides an agent-based workflow similar to Claude Code or Cursor, but entirely offline and integrated inside IntelliJ IDEA or any JetBrains IDE.

## Quick links

- **[Getting Started](getting-started/)** — install the plugin, point it at a model, start chatting
- **[Agent & Tools](agent-and-tools/)** — how the agent loop works and what tools are available
- **[MCP Servers](mcp-servers/)** — connect any MCP-compatible tool server to the agent
- **[External Knowledge](external-knowledge/)** — web search, docs research, and error analysis (1.5.0+)
- **[Code Health Monitor](code-health-monitor/)** — background code quality analysis (1.5.0+)
- **[Settings Reference](settings-reference/)** — full configuration reference
- **[Changelog](changelog/)** — version history from 1.2.0 to 1.5.0

## Key features

- **Project-aware chat** — Ask questions about your codebase with persistent, multi-session support
- **Local AI models** — Run models fully offline via GGUF; your code never leaves your machine
- **AI agent for code editing** — Generate and modify code using natural language with safe diff previews
- **Context-aware assistance** — Understands the current file, selection, git state, and project structure
- **MCP server support** — Connect any MCP-compatible tool server (1.4.0+)
- **External Knowledge** — Research the web, official docs, and GitHub issues from inside the IDE (1.5.0+)
- **Code Health Monitor** — Proactive background analysis for regressions, complexity, and refactor opportunities (1.5.0+)
- **Multi-task agent** — Run multiple agent tasks in parallel with a live task queue panel (Pro)
- **Aggressive mode** — Let the agent act decisively without prompting on each step (Pro)

## Free vs Pro

| Capability | Free | Pro |
|---|---|---|
| Chat (Ask mode) | Unlimited | Unlimited |
| Single-session agent | Yes | Yes |
| Multi-task agent | — | Yes |
| MCP servers | 1 (manual approval) | Unlimited + auto-approval |
| Aggressive mode | — | Yes |
| Multi-file apply | — | Yes |
| External Knowledge | 5 searches/day, 1 docs source | 50 searches/day, 10 sources |
| Code Health Monitor | LIGHT / manual only | BALANCED + AGGRESSIVE + background |

Upgrade at [llamatik.com](https://www.llamatik.com).
