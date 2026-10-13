---
title: "Agent Framework (v2.0)"
weight: 14
---

`com.llamatik.sdk.framework` is a complete Kotlin DSL for building agentic AI applications.
It runs on all Llamatik targets: Android, iOS, desktop, and WASM.

Add it:

```kotlin
implementation("com.llamatik:sdk:<version>")
```

## Creating an agent

Use the `Agent { }` builder to declare an agent:

```kotlin
import com.llamatik.sdk.framework.*

val agent = Agent {
    name("Kotlin Assistant")
    systemPrompt("You are an expert Kotlin developer.")
    temperature(0.7f)
    maxTokens(1024)
}
```

## Streaming

`agent.stream()` returns a `Flow<AgentStreamChunk>`:

```kotlin
agent.stream("Explain coroutines.").collect { chunk ->
    when (chunk) {
        is AgentStreamChunk.Delta           -> print(chunk.text)
        is AgentStreamChunk.ToolCallStarted -> println("[tool: ${chunk.toolId}]")
        is AgentStreamChunk.ToolResult      -> println("[result: ${chunk.summary}]")
        is AgentStreamChunk.Done            -> println("\nDone.")
        is AgentStreamChunk.Failure         -> println("Error: ${chunk.message}")
    }
}
```

`agent.run(prompt)` collects the stream to a single `String` for non-streaming use cases.

## Memory

| Type | Description |
|---|---|
| `ConversationMemory()` | Unbounded in-memory conversation history (default) |
| `SlidingWindowMemory(windowSize)` | Keeps the last N messages; always pins the system message |
| `PersistentMemory(storage, key)` | Survives restarts; implement `MemoryStorage` for your platform |
| `SummaryMemory(triggerSize, compressCount) { ... }` | Compresses old messages via a summarizer when the window fills |
| `NoMemory` | Stateless — each turn is independent |

```kotlin
val agent = Agent {
    memory(SlidingWindowMemory(windowSize = 20))
}

// PersistentMemory — restore a prior session on startup
val agent = Agent {
    persistentMemory(storage = myStorage, key = "sessions/chat.txt")
}
// agent.memory.restore()  // call after construction to reload prior session
// agent.memory.persist()  // call after each turn you want to save
```

## Built-in tools

Register built-in tools with `builtIns()`:

```kotlin
val agent = Agent {
    systemPrompt("You are a helpful assistant.")
    tools { builtIns() }  // Calculator, DateTime, UUID, Random
}
```

Or add individual tools:

```kotlin
val agent = Agent {
    tools {
        calculator()
        datetime()
        uuid()
        random()
    }
}
```

## Custom tools

Implement `AgentTool` (also aliased as `Tool`):

```kotlin
class WeatherTool : AgentTool {
    override val id = "weather"
    override val displayName = "Weather"
    override val description = "Returns the current weather for a city."
    override val schema = buildJsonObject {
        put("type", "object")
        putJsonObject("properties") {
            putJsonObject("city") { put("type", "string") }
        }
        putJsonArray("required") { add("city") }
    }
    override fun isSupported() = true

    override suspend fun execute(input: JsonObject): AgentToolResult {
        val city = input["city"]?.jsonPrimitive?.content
            ?: return AgentToolResult.Failure("Missing city")
        return AgentToolResult.Success("Sunny, 22°C in $city")
    }
}

val agent = Agent {
    tools { register(WeatherTool()) }
}
```

## Observable events

Subscribe to turn-lifecycle events via `agent.events`:

```kotlin
agent.events.collect { event ->
    when (event) {
        is AgentEvent.TurnStarted     -> println("Turn: ${event.userMessage}")
        is AgentEvent.Thinking        -> println("Thinking…")
        is AgentEvent.ToolCallStarted -> println("Calling ${event.toolId}")
        is AgentEvent.ToolCallEnded   -> println("Tool done")
        is AgentEvent.TurnCompleted   -> println("Done: ${event.response}")
        is AgentEvent.Failed          -> println("Error: ${event.message}")
    }
}
```

## Workflows

Chain agent invocations into a sequential workflow:

```kotlin
val workflow = Workflow {
    step("outline") { _ ->
        "Create an outline for an article about Kotlin coroutines."
    }
    step("write") { ctx ->
        val outline = ctx["outline"] ?: ""
        "Write a full article based on:\n$outline"
    }
    step("review") { ctx ->
        val article = ctx["write"] ?: ""
        "Review this article for clarity:\n$article"
    }
}

val result = workflow.execute(agent)
println(result.finalOutput)
```

### Approval gates

Use `gatedStep` to pause and ask for confirmation before proceeding:

```kotlin
val workflow = Workflow {
    step("research") { _ -> "Research the topic: Kotlin coroutines." }

    gatedStep("plan", gate = { stepName, proposal ->
        userConfirms(stepName, proposal)  // return true to proceed, false to stop
    }) { ctx ->
        "Create an implementation plan based on:\n${ctx["research"]}"
    }

    step("implement") { ctx ->
        "Implement according to plan:\n${ctx["plan"]}"
    }
}
```

## Prompt pipeline

The framework processes each turn through a staged pipeline:

1. **SystemStage** — injects the system prompt
2. **MemoryStage** — adds conversation history from the active memory
3. **RagStage** — retrieves and injects relevant context (when configured)
4. **UserMessageStage** — appends the current user message

## Package reference

| Type | Package |
|---|---|
| `Agent`, `AgentConfig` | `com.llamatik.sdk.framework` |
| `Memory`, `ConversationMemory`, `SlidingWindowMemory` | `com.llamatik.sdk.framework` |
| `PersistentMemory`, `MemoryStorage`, `SummaryMemory` | `com.llamatik.sdk.framework` |
| `PromptPipeline`, `PromptStage` | `com.llamatik.sdk.framework` |
| `AgentEvent`, `AgentStreamChunk` | `com.llamatik.sdk.framework` |
| `Workflow`, `Step`, `WorkflowResult` | `com.llamatik.sdk.framework` |
| `Tool` (= `AgentTool`) | `com.llamatik.sdk.framework` |
| `CalculatorTool`, `DateTimeTool`, `UuidTool`, `RandomTool` | `com.llamatik.sdk.agent` |
