---
title: "LlamaSession"
weight: 6
---

`LlamaSession` is an isolated inference session with its own KV cache and cancel flag.
It is the right tool when you need multiple independent generation streams running concurrently.

Create sessions via `LlamaBridge.createSession()`. Dispose with `close()` when done.

## Platform support

- **Android**: supported
- **iOS**: supported
- **JVM / Desktop**: supported
- **WASM**: currently not implemented — `createSession()` returns `null`

## Public API

```kotlin
expect class LlamaSession {
    fun stream(prompt: String, callback: GenStream)
    fun cancel()
    fun close()
}
```

## Creating a session

Sessions are created from `LlamaBridge` after a generation model has been loaded:

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val session = LlamaBridge.createSession()
    ?: error("model not loaded or sessions not supported on this platform")
```

Each call to `createSession()` returns a new independent session.

## `stream(prompt, callback)`
Streams tokens for the given prompt into the callback.
This call blocks until the stream finishes or is cancelled.
Always call it from a background thread.

```kotlin
session.stream(
    prompt = "Explain Kotlin Flows in simple terms.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## `cancel()`
Cancels an in-progress `stream` call for this session.
Other sessions running concurrently are not affected.

```kotlin
session.cancel()
```

## `close()`
Releases the native resources associated with this session.
Do not call while `stream` is still running.

```kotlin
session.close()
```

## Concurrent sessions example

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val sessionA = LlamaBridge.createSession() ?: error("not supported")
val sessionB = LlamaBridge.createSession() ?: error("not supported")

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

Each session maintains its own KV state.
Sessions do not share context — `sessionA`'s history does not affect `sessionB`.

## When to use sessions vs the global bridge

| Scenario | Recommendation |
|---|---|
| Single-user chat interface | `LlamaBridge.generateStream(...)` |
| Side-by-side generation (compare two prompts) | `LlamaSession` ×2 |
| Batched per-user inference in a server process | One `LlamaSession` per user |
| Background summarization while chat is active | `LlamaSession` for background task |
