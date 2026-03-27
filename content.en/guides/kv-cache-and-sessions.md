---
title: "KV cache and sessions"
weight: 6
---

KV cache reuse is one of the most important performance features for chat-like experiences.
Instead of rebuilding the full prompt state from scratch for every turn, the model can continue from what it has already processed.

## Core methods

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

## Important limitations

- Session persistence is currently unavailable on WASM.
- Session files should be used with the same model setup that created them.
- If no active session exists, `generateContinue(...)` falls back to fresh generation behavior.

## When this feature is worth using

Use KV sessions when:
- you are building a chat app
- the user asks follow-up questions
- you want faster continuation across turns

Skip it when:
- every request is independent
- you deliberately want each run to start with a clean state
