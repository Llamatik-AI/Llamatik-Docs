---
title: "Code Health Monitor"
weight: 5
---

# Code Health Monitor

The Code Health Monitor is a proactive background analysis layer that continuously inspects your codebase for structural problems, regression risks, and refactoring opportunities — without requiring a model call for every insight.

*Available since version 1.5.0.*

## Enabling it

1. Open **Settings → Llamatik Code → Code Health Monitor**.
2. Toggle **Enable Code Health Monitor**.
3. Choose an analysis mode and run trigger.

## Analysis modes

| Mode | Files per scan | Max runtime | Model calls/hour | Who |
|---|---|---|---|---|
| `LIGHT` | 5 | 8 s | 3 | Free + Pro |
| `BALANCED` | 30 | 30 s | 20 | Pro only |
| `AGGRESSIVE` | 150 | 120 s | 50 | Pro only |

Free users are always capped at LIGHT mode regardless of settings.

## Run triggers

| Trigger | When it fires | Who |
|---|---|---|
| `MANUAL` | Only when you explicitly request a scan | Free + Pro |
| `ON_SAVE` | 3 seconds after a file is saved (debounced) | Pro only |
| `IDLE` | When the IDE has been idle | Pro only |
| `BEFORE_COMMIT` | Before a git commit | Pro only |

## Static analysis (no model required)

| Check | Trigger | Severity |
|---|---|---|
| Large file | >300 lines | WARNING |
| TODO/FIXME accumulation | ≥3 occurrences in a file | INFO |
| Empty catch block | `catch (...) {}` | WARNING |
| Deprecated API usage | `@Deprecated` present | INFO |

## Regression risk detection

When a git diff is present, the analyser scans it for regression patterns:

| Pattern | Severity |
|---|---|
| Removed public function / class / interface | HIGH |
| Deleted `@Test` annotation | HIGH |
| Changed Gradle dependency | WARNING |
| Removed `try` or `catch` block | WARNING |
| Modified `AndroidManifest.xml` | WARNING |

## Refactor opportunity detection

- **Long functions** — functions exceeding 40 lines
- **Repeated catch patterns** — four or more identical catch blocks in a file

## Insight severity

| Severity | Meaning | Notification |
|---|---|---|
| `HIGH` | Likely regression or broken callers | Balloon notification |
| `WARNING` | Worth reviewing before release | Insights panel only |
| `INFO` | Low-priority improvement | Insights panel only |

## Watched areas

Pin paths to prioritise their insights:

| Tool | Description |
|---|---|
| `watch_area_add` | Add a file, directory, package, or module |
| `watch_area_remove` | Remove a watched area by ID |
| `watch_area_list` | List all watched areas |

| | Free | Pro |
|---|---|---|
| Watched areas | 1 | 20 |

## Before-commit assistant

Run on demand or automatically before every commit (Pro). Produces:
- Lines added/removed across all staged files
- A suggested commit message
- All regression-risk and dependency-risk insights from the diff
- A flag if any watched areas are affected

## All Code Health agent tools

| Tool | Description |
|---|---|
| `code_health_status` | Current monitor status, mode, and last scan time |
| `code_health_refresh` | Trigger an immediate scan |
| `code_health_list_insights` | List all insights, filtered by severity or type |
| `code_health_explain_insight` | Detailed explanation with session context |
| `code_health_create_fix_plan` | Step-by-step fix plan for an insight |
| `code_health_ignore_insight` | Suppress an insight by ID |
| `watch_area_add` | Add a watched area |
| `watch_area_remove` | Remove a watched area |
| `watch_area_list` | List watched areas |
| `session_context_get` | Read the current session context |
| `session_context_clear` | Reset the session context |
| `before_commit_review` | Review staged changes for regression risks |
| `regression_risk_analyze` | Analyse a git diff for regression patterns |
| `refactor_analyze_file` | Analyse a file for refactoring opportunities |
