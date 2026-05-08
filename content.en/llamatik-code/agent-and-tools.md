---
title: "Agent & Tools"
weight: 2
---

# Agent & Tools

The Llamatik Code agent is a tool-driven loop that takes a natural-language instruction, plans steps, calls built-in tools, and applies changes to your project with a safe diff preview.

## How the agent works

1. You type an instruction in **Agent mode**.
2. The agent makes a first LLM pass to produce a plan.
3. It executes each step by calling one or more tools.
4. Results are fed back into the loop until the task is complete.
5. File writes are shown as a **diff preview** — you approve before changes land on disk.
6. After apply, the agent optionally triggers a build and interprets errors.

## Built-in tools (available since 1.2.0)

| Tool | Description |
|---|---|
| `read_file` | Read the content of any project file |
| `write_file` | Write or overwrite a file (always previewed) |
| `list_files` | List files in a directory |
| `search_code` | Full-text search across the project |
| `run_terminal` | Execute a shell command |
| `get_diagnostics` | Fetch current IDE inspection/compiler errors |
| `explain_build_failure` | Interpret the last build failure |
| `run_diagnostics` | Run plugin self-diagnostics |

## Git tools (1.3.0+)

| Tool | Description |
|---|---|
| `git_diff` | Diff of current changes (staged + unstaged) |
| `git_status` | Working-tree status |
| `git_log` | Recent commit history |
| `git_blame` | Line-level blame for a file |
| `git_changed_files` | Files changed in the current diff |

## Project intelligence tools (1.3.0+)

| Tool | Description |
|---|---|
| `index_search_files` | Fast file search by name pattern |
| `index_search_symbols` | Symbol search across the project index |
| `index_list_recent` | Recently opened or modified files |
| `psi_find_usages` | Find all usages of a symbol via PSI |
| `psi_get_type` | Resolve the type of an expression |
| `psi_get_declarations` | List declarations in a file |
| `smart_search` | Fuzzy search ranked by recency and relevance |

## Codebase rules (1.3.0+)

Write persistent natural-language rules that the agent follows in every session:

| Tool | Description |
|---|---|
| `rules_get` | Read the current ruleset |
| `rules_set` | Replace the ruleset with new content |
| `rules_clear` | Delete all rules |

Example rule: `"Always use Result<T> for error returns. Never throw exceptions in repository classes."`

## Run config tools (1.3.0+)

| Tool | Description |
|---|---|
| `run_config_list` | List available IDE run configurations |
| `run_config_execute` | Trigger a run configuration |

## KMP native tools (1.3.0+)

| Tool | Description |
|---|---|
| `kmp_diagnose` | Detect Kotlin Multiplatform native compilation issues |
| `kmp_fix_plan` | Generate a structured fix plan for KMP native errors |

## Agent trace panel

Every tool call is recorded in the **What I tried** trace sidebar:

- Tool name and arguments
- Duration in milliseconds
- Success / error result
- MCP server name (for MCP tools)

## Context actions

Right-click any selection or file for quick one-shot operations:

- **Ask About Selection** — ask a question about selected code
- **Explain This File** — explain the entire current file
- **Refactor Selection** — refactor selected code with a diff preview
- **Fix Selected Code** — fix selected code
- **Generate Tests** — generate a test class for the current file
- **Add Documentation** — generate KDoc/Javadoc
- **Edit with Prompt** — apply any instruction to the selection
- **Explain Failing Build** — interpret the last build error
- **Explain Current Git Changes** — explain the current diff and suggest a commit message
