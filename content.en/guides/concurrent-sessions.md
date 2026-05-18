---
title: "Concurrent sessions"
weight: 10
---

`LlamaSession` lets you run multiple independent generation streams at the same time.
Each session has its own KV cache and cancel flag, so they do not interfere with one another.

## When to use sessions

The global `LlamaBridge.generateStream(...)` works well for single-threaded chat flows.
Use `LlamaSession` when:

- you want to compare responses to the same prompt from different system prompts side by side
- you have multiple users sharing a single JVM process
- you need a background task (summarization, classification) while the main chat is active

## Create a session

Sessions are created from `LlamaBridge` after the generation model is loaded.
Pass a `name` string to give the session a human-readable label:

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val session = LlamaBridge.createSession(name = "My chat")
    ?: error("sessions not supported on this platform")

println(session.name) // "My chat"
```

The `name` parameter is optional and defaults to an empty string.
Each call returns a new independent session with a fresh KV cache.

## Stream from a session

`session.stream(...)` blocks until the stream finishes.
Call it from a background thread:

```kotlin
session.stream(
    prompt = "What is Kotlin Multiplatform?",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Concurrent example

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val sessionA = LlamaBridge.createSession(name = "Forest") ?: error("not supported")
val sessionB = LlamaBridge.createSession(name = "City") ?: error("not supported")

val threadA = thread {
    sessionA.stream("Describe a forest at dawn.", callbackA)
}
val threadB = thread {
    sessionB.stream("Describe a city at night.", callbackB)
}

threadA.join()
threadB.join()

sessionA.close()
sessionB.close()
```

Both streams run in parallel. Their KV states are completely isolated.

## Cancelling a session

Cancel only the stream for that specific session:

```kotlin
session.cancel()
```

Other sessions are not affected.

## Closing a session

Always close a session when you are done with it:

```kotlin
session.close()
```

Do not call `close()` while `stream()` is still running on the session.

## Comparison with the global bridge

| | Global bridge | `LlamaSession` |
|---|---|---|
| Concurrent streams | No | Yes |
| Shared KV cache | Yes | No (each session is isolated) |
| Cancel one without stopping others | No | Yes |
| WASM support | Yes | Not currently |

## Platform notes

- Sessions are fully supported on Android, iOS, and JVM.
- On WASM, `createSession()` returns `null` — use the global bridge streaming API instead.
