---
title: "Settings Reference"
weight: 6
---

# Settings Reference

All Llamatik Code settings live under **Settings → Llamatik Code**.

## Llamatik Code (main page)

| Setting | Description |
|---|---|
| License key | Pro license key |
| Trial status | Days remaining in free trial |

## Inference

| Setting | Description | Default |
|---|---|---|
| Model path | Absolute path to a GGUF model file | — |
| Context size | Token context window | 4096 |
| Threads | CPU threads for inference | System cores / 2 |
| GPU layers | Layers offloaded to GPU (0 = CPU only) | 0 |
| Temperature | Sampling temperature | 0.7 |
| Top-p | Nucleus sampling threshold | 0.9 |
| Agent fallback policy | ABORT / RETRY / ESCALATE on step failure | ABORT |
| Run benchmark | Measure tokens/s for the current model | — |

## MCP Servers *(1.4.0+)*

| Setting | Description |
|---|---|
| Server list | All configured MCP servers |
| Add server | Open the server editor |
| Auto-approval rules | Per-tool and per-server rules (Pro) |

Per-server fields: name, command, arguments, environment variables, working directory, enabled, auto-start, trust level.

## External Knowledge *(1.5.0+)*

| Setting | Description | Default |
|---|---|---|
| Enable external knowledge | Master toggle | off |
| Max searches per day | Daily quota (up to tier limit) | tier default |
| Max pages per task | Pages fetched per task | tier default |
| Enable auto-use | Agent searches automatically (Pro) | off |
| Docs sources | Private documentation sources | empty |

Tier limits: Free — 5 searches/day, 3 pages/task, 1 docs source. Pro — 50/day, 10/task, 10 sources.

## Code Health Monitor *(1.5.0+)*

| Setting | Description | Default |
|---|---|---|
| Enable Code Health Monitor | Master toggle | off |
| Analysis mode | OFF / LIGHT / BALANCED / AGGRESSIVE | OFF |
| Run trigger | MANUAL / ON_SAVE / IDLE / BEFORE_COMMIT | MANUAL |
| Max files per scan | Override mode file budget | mode default |
| Max runtime (ms) | Override mode time budget | mode default |
| Max model calls per hour | Rate limit for model-backed insights | mode default |
| Live suggestions | Inline hints as you type (Pro) | off |
| Test awareness | Include missing-test detection | on |
| Regression detection | Include git-diff analysis | off |
| Show dashboard | Enable health dashboard panel | on |
| Before-commit review | Auto-run before every commit (Pro) | off |

### Budget reference

| Mode | Max files | Max runtime | Model calls/hour | Max insights/file |
|---|---|---|---|---|
| LIGHT | 5 | 8 s | 3 | 3 |
| BALANCED (Pro) | 30 | 30 s | 20 | 10 |
| AGGRESSIVE (Pro) | 150 | 120 s | 50 | 20 |

## Autonomous Local Development *(1.8.0+)*

Settings for the Offline Enterprise Pair Programmer and inline completion features.

### Enterprise Pair Programmer

| Setting | Description | Default |
|---|---|---|
| Enable Autonomous Local Development | Master toggle for the entire mode | on |
| Enable Offline Enterprise Pair Programmer | Enable the enterprise mission workflow | on |
| Default safety level | APPROVAL_REQUIRED / PLAN_ONLY / AUTO_APPLY / FULL_AUTO | APPROVAL_REQUIRED |
| Allow external knowledge during missions | Let the agent search the web during enterprise missions | off |
| Require plan before writing files | Block file writes until a mission plan exists | on |
| Require diff preview before applying patches | Show unified diff before every write | on |
| Require audit report on mission completion | Produce an audit report when the mission closes | on |

### Inline Completions

| Setting | Description | Default |
|---|---|---|
| Enable inline completions | Show AI suggestions as you type | off |
| Debounce delay (ms) | Idle time before a completion request is sent | 600 ms |
| Max context chars | Characters of prefix + suffix sent to the model | 1200 |
| Max completion tokens | Maximum tokens in the generated completion | 64 |
