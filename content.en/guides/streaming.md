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

## Lifecycle tips

- Cancel generation when the user navigates away.
- Buffer deltas and render efficiently.
- Do not start multiple heavy streams on the same small device unless you have tested the behavior carefully.
- On WASM, prefer streaming APIs when running in worker-only mode.
