---
title: "External Knowledge"
weight: 4
---

# External Knowledge

External Knowledge gives the Llamatik Code agent the ability to search the web, read documentation pages, research errors, and query private docs sources — all from inside the IDE, with citations.

*Available since version 1.5.0.*

## Enabling it

1. Open **Settings → Llamatik Code → External Knowledge**.
2. Toggle **Enable external knowledge**.

## Web search

The agent searches the web via DuckDuckGo — no API key required.

| Tool | Description |
|---|---|
| `external_search` | General web search |
| `external_search_official_docs` | Search restricted to official documentation sites |
| `external_search_github_issues` | Search GitHub Issues for known bugs and solutions |

| | Free | Pro |
|---|---|---|
| Searches per day | 5 | 50 |
| Results per search | 5 | 10 |

## Page reading

| Tool | Description |
|---|---|
| `external_read_page` | Fetch and read a single URL |
| `external_read_pages` | Read multiple URLs in parallel |

| | Free | Pro |
|---|---|---|
| Pages per task | 3 | 10 |

## Research orchestration

The research orchestrator detects the intent behind a query and plans a multi-step research workflow automatically.

| Intent | Triggered by | Secondary queries added |
|---|---|---|
| `LIBRARY_DOCS` | Library or framework name | "documentation" |
| `BUG_ERROR` | Stack trace, exception, error code | "error fix" |
| `API_BEHAVIOR` | API name + how/does/when | "API example" |
| `MIGRATION` | migrate, upgrade, from X to Y | "migration guide" |
| `LATEST_VERSION` | latest, current version, release | "latest version release" |

| Tool | Description |
|---|---|
| `research_plan` | Plan a multi-step research workflow |
| `research_execute` | Execute the planned queries |
| `research_summarize` | Synthesise findings into an answer with citations |
| `research_route_sources` | Route a query to the most relevant source types |
| `research_fix_plan` | Produce a fix plan from research findings |

## Smart source routing

| Keyword in query | Routes to |
|---|---|
| `kotlin`, `coroutine`, `suspend` | kotlinlang.org |
| `android`, `compose`, `jetpack` | developer.android.com |
| `ios`, `swift`, `swiftui`, `xcode` | developer.apple.com |
| `spring`, `springboot` | spring.io |
| `bug`, `issue`, `error`, `crash` | github.com |
| default | DuckDuckGo |

## Error research

| Tool | Description |
|---|---|
| `error_research_analyze` | Analyse an error; detect build tool, framework, and error category |
| `error_research_fix` | Generate a structured fix plan for the analysed error |

The fix plan includes the error category, likely root cause, ordered fix steps with code suggestions, and links to relevant documentation.

## Private docs sources

Add internal wikis, SDK docs, or API references for the agent to search.

| Tool | Description |
|---|---|
| `docs_sources_list` | List all configured docs sources |
| `docs_source_add` | Add a docs source by URL |
| `docs_index_refresh` | Re-crawl and re-index all docs sources |
| `docs_search` | Search across all indexed docs sources |

| | Free | Pro |
|---|---|---|
| Docs sources | 1 | 10 |
| Auto-use in agent | — | Yes |

## Citations

Every research result backed by external content includes structured citations:

```
[1] kotlinlang.org — Kotlin Coroutines: Basics (official documentation, high confidence)
    https://kotlinlang.org/docs/coroutines-basics.html
```

Citation source types: `OFFICIAL_DOCS`, `GITHUB_ISSUES`, `STACK_OVERFLOW`, `BLOG`, `RELEASE_NOTES`, `CHANGELOG`, `PACKAGE_REGISTRY`.

## Editor action

Right-click any selected text → **Search Docs & Web** to open a pre-filled search.
