---
title: "Agent Platform (v2.0)"
weight: 15
---

`com.llamatik:sdk-agent` is the Agent Platform runtime — a production-ready runtime layer on top of the Agent Framework that adds permissions, companion profiles, audit logging, persistent memory, and a workflow engine.

Add it:

```kotlin
implementation("com.llamatik:sdk-agent:<version>")
```

`sdk-agent` depends on `sdk`, so you do not need to declare both when using the platform.

## Architecture

```
┌──────────────────────────────────────┐
│           Your application           │
└───────────────┬──────────────────────┘
                │
┌───────────────▼──────────────────────┐
│           AgentRuntime               │
│  • processMessage()                  │
│  • CompanionProfile management       │
│  • PermissionGate enforcement        │
│  • AuditLog recording                │
│  • WorkflowEngine execution          │
│  • Persistent AgentMemory            │
└───────────────┬──────────────────────┘
                │
┌───────────────▼──────────────────────┐
│   Agent (sdk framework DSL)          │
│   ToolRegistry  │  BuiltInTools      │
└───────────────┬──────────────────────┘
                │
┌───────────────▼──────────────────────┐
│   com.llamatik:sdk   │   :core       │
└──────────────────────────────────────┘
```

## Creating an AgentRuntime

Use the builder to configure the runtime:

```kotlin
import com.llamatik.sdk.agent.runtime.*
import com.llamatik.sdk.agent.tools.*

val runtime = AgentRuntime.Builder()
    .profile(CompanionProfile.General)
    .enableAuditLog(storage = myStorage)
    .permissions(PermissionSet.Default)
    .memory(AgentMemory.persistent(storage = myStorage, key = "agent/memory"))
    .tools {
        all()  // registers all 9 built-in tools
    }
    .build()
```

## Processing a message

```kotlin
val response: AgentResponse = runtime.processMessage(
    userMessage = "What files are in my downloads folder?",
    context = AgentContext(sessionId = "session-1"),
)

when (response) {
    is AgentResponse.Reply   -> println(response.text)
    is AgentResponse.Blocked -> println("Blocked: ${response.reason}")
    is AgentResponse.Failed  -> println("Error: ${response.message}")
}
```

Streaming:

```kotlin
runtime.streamMessage("Summarise my notes.").collect { chunk ->
    when (chunk) {
        is AgentStreamChunk.Delta  -> print(chunk.text)
        is AgentStreamChunk.Done   -> println()
        else                       -> Unit
    }
}
```

## Companion profiles

A `CompanionProfile` presets the system prompt, tool allowlist, memory window, and tone:

| Profile | Description |
|---|---|
| `CompanionProfile.General` | Balanced assistant; all tools allowed |
| `CompanionProfile.Developer` | Code-focused; file/shell tools emphasised |
| `CompanionProfile.Writer` | Writing assistant; web search enabled |
| `CompanionProfile.Analyst` | Data analysis focus; calculator and web tools |
| `CompanionProfile.Custom(config)` | Fully customised via `CompanionConfig` |

```kotlin
val runtime = AgentRuntime.Builder()
    .profile(CompanionProfile.Developer)
    .build()
```

## Permissions

`PermissionGate` intercepts tool calls and enforces the active `PermissionSet`:

```kotlin
val runtime = AgentRuntime.Builder()
    .permissions(
        PermissionSet.builder()
            .allow(ToolId.Calculator)
            .allow(ToolId.DateTime)
            .deny(ToolId.FileSystem)    // block file access
            .requireApproval(ToolId.WebSearch)  // prompt user each time
            .build()
    )
    .build()
```

Denied tools return `AgentResponse.Blocked`; tools that require approval trigger the optional `onApprovalRequired` callback you register on the builder.

## Built-in tools

| Tool | ID | Description |
|---|---|---|
| `CalculatorTool` | `calculator` | Safe arithmetic expression evaluator |
| `DateTimeTool` | `datetime` | Current date/time, timezone conversions |
| `UuidTool` | `uuid` | UUID generation |
| `RandomTool` | `random` | Bounded random numbers |
| `WebSearchTool` | `web_search` | Web search via Llamatik Search API |
| `WebFetchTool` | `web_fetch` | Fetch and parse a URL |
| `FileSystemTool` | `file_system` | Read/write sandboxed filesystem paths |
| `TranslationTool` | `translation` | Text translation (local model) |
| `SummarizerTool` | `summarizer` | Summarise long text via LLM |

```kotlin
val runtime = AgentRuntime.Builder()
    .tools {
        all()                    // register all 9 tools
        // or selectively:
        calculator()
        datetime()
        webSearch(apiKey = key)
        fileSystem(sandbox = myDir)
    }
    .build()
```

## Custom tool registration

```kotlin
class MyTool : AgentTool { ... }

val runtime = AgentRuntime.Builder()
    .tools {
        register(MyTool())
    }
    .build()
```

## Audit log

Every tool call and response is recorded to the audit log when enabled:

```kotlin
val runtime = AgentRuntime.Builder()
    .enableAuditLog(storage = myStorage)
    .build()

// Query the log later
val entries: List<AuditEntry> = runtime.auditLog.queryRecent(limit = 50)
```

Each `AuditEntry` records: `sessionId`, `timestamp`, `toolId`, `input`, `output`, `durationMs`, `blocked`.

## Workflow engine

The embedded `WorkflowEngine` runs `Workflow` instances (defined with the Agent Framework DSL) with full permission and audit integration:

```kotlin
val workflow = Workflow {
    step("search")  { _ -> "Search for recent Kotlin releases." }
    step("analyse") { ctx -> "Analyse: ${ctx["search"]}" }
    step("report")  { ctx -> "Write a report based on:\n${ctx["analyse"]}" }
}

val result: WorkflowResult = runtime.runWorkflow(workflow)
println(result.finalOutput)
```

## Persistent memory

`AgentMemory` persists conversation context and key-value facts across sessions:

```kotlin
// Builder
val runtime = AgentRuntime.Builder()
    .memory(AgentMemory.persistent(storage = myStorage, key = "agent/mem"))
    .build()

// Manual save / restore
runtime.memory.persist()
runtime.memory.restore()

// Store a named fact
runtime.memory.remember("user_name", "Alice")
val name = runtime.memory.recall("user_name")
```
