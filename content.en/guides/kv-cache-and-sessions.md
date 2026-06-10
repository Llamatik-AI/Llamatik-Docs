---
title: "KV cache and sessions"
weight: 6
---

KV cache reuse is one of the most important performance features for chat-like experiences.
Instead of rebuilding the full prompt state from scratch for every turn, the model can continue from what it has already processed.

There are two ways to use the KV cache:

- **`LlamaSession`** — for streaming multi-turn conversations. Each session owns an isolated KV cache that persists across `stream()` calls. This is the recommended approach for chat UIs.
- **Global session methods** — for non-streaming multi-turn generation via `generateContinue()`, with optional disk persistence via `sessionSave()`/`sessionLoad()`.

## Core methods (global, non-streaming)

```kotlin
LlamaBridge.sessionReset()
LlamaBridge.sessionSave(path)
LlamaBridge.sessionLoad(path)
LlamaBridge.generateContinue(prompt)
```

## Fresh turn vs continued turn

### Fresh generation

```kotlin
val answer = LlamaBridge.generate("Explain Kotlin coroutines.")
```

Use this when the request is independent and you do not care about previous turns.

### Continued generation

```kotlin
val answer2 = LlamaBridge.generateContinue("Now show a short example.")
```

Use this when you want the next prompt to continue from the current session state.

## Typical chat flow

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val first = LlamaBridge.generate("Explain Kotlin coroutines in simple words.")
val second = LlamaBridge.generateContinue("Now give one practical example.")
val third = LlamaBridge.generateContinue("Summarize both answers in 3 bullets.")
```

## Saving a session

```kotlin
val sessionPath = "/tmp/demo.session"
val saved = LlamaBridge.sessionSave(sessionPath)
check(saved)
```

This lets you persist the current conversation state across app restarts or later reuse.

## Loading a session

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val loaded = LlamaBridge.sessionLoad(sessionPath)
check(loaded)

val resumed = LlamaBridge.generateContinue("Continue from where we stopped.")
```

## Resetting the session

```kotlin
LlamaBridge.sessionReset()
```

This is useful when the user starts a new conversation but you want to keep the model loaded.

## Streaming multi-turn chat with KV continuity

For streaming conversations, use `LlamaSession` instead of the global methods.
Each `session.stream(...)` call appends the new prompt tokens to the session's existing KV cache,
so the model retains full memory of previous turns.

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val session = LlamaBridge.createSession(name = "chat")
    ?: error("sessions not supported on this platform")

// Turn 1
session.stream("What is Kotlin?", callback)

// Turn 2 — model remembers what was said in turn 1
session.stream("Give me an example of a coroutine.", callback)

// Explicitly reset the KV state when the conversation is over
session.close()
```

The global `LlamaBridge.generateStream(...)` is stateless and does not carry KV context across calls.
Use `LlamaSession` whenever the model must remember previous turns while streaming.

See [Concurrent Sessions]({{< relref "concurrent-sessions" >}}) for running multiple independent sessions in parallel.

## Important limitations

- Session persistence (`sessionSave`/`sessionLoad`) is currently unavailable on WASM.
- Session files must be used with the same model that created them.
- If no active session exists, `generateContinue(...)` falls back to fresh generation behavior.
- `LlamaSession` is not supported on WASM — use `LlamaBridge.generateStream(...)` there instead.

## When this feature is worth using

Use KV sessions when:
- you are building a chat app
- the user asks follow-up questions
- you want faster continuation across turns
- you are streaming and need the model to remember previous turns

Skip it when:
- every request is independent
- you deliberately want each run to start with a clean state
