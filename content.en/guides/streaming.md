---
title: "Streaming"
weight: 3
---

Streaming lets your app consume generation output incrementally.
This usually feels much better than waiting for the full answer, especially on-device where responses may take time.

## Plain streaming

```kotlin
LlamaBridge.generateStream(
    prompt = "Stream a short poem.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Streaming with context

```kotlin
LlamaBridge.generateStreamWithContext(
    systemPrompt = "You are concise.",
    contextBlock = "Topic: on-device LLMs.",
    userPrompt = "Give me 3 bullet points.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Streaming JSON

```kotlin
LlamaBridge.generateJsonStream(
    prompt = "Return a JSON object for one task.",
    jsonSchema = schema,
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Streaming with KV cache continuity

A common issue was that streaming across multiple turns caused the model to lose memory — the KV cache was being reset on every call.
The correct pattern is to use `LlamaSession` for streaming multi-turn conversations:

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val session = LlamaBridge.createSession(name = "chat")
    ?: error("sessions not supported on this platform")

// Turn 1 — KV state is built from this prompt
session.stream(
    prompt = "Explain Kotlin coroutines.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println()
        override fun onError(message: String) = println("Error: $message")
    }
)

// Turn 2 — continues from the existing KV state; model remembers the previous turn
session.stream(
    prompt = "Now show a short example.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println()
        override fun onError(message: String) = println("Error: $message")
    }
)

session.close()
```

Each `session.stream(...)` call appends its prompt tokens to the session's KV cache and generates from there.
The global `LlamaBridge.generateStream(...)` is stateless by design and does not carry KV state across calls.
Use `LlamaSession` whenever you need the model to remember previous turns during streaming.

## Lifecycle tips

- Cancel generation when the user navigates away.
- Buffer deltas and render efficiently.
- Use `LlamaSession` for multi-turn streaming; use `LlamaBridge.generateStream(...)` only for independent one-shot requests.
- Do not start multiple heavy streams on the same small device unless you have tested the behavior carefully.
- On WASM, prefer streaming APIs when running in worker-only mode.
