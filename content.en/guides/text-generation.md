---
title: "Text generation"
weight: 2
---

Text generation is the core `LlamaBridge` workflow.

## Basic one-shot generation

Use `generate(...)` when each request is independent.

```kotlin
val ok = LlamaBridge.initGenerateModel(modelPath)
check(ok)

val answer = LlamaBridge.generate("Explain Llamatik in one sentence.")
println(answer)
```

This is the right choice for:
- one-off prompts
- utility screens
- simple summarization tasks
- commands that do not need chat history

## Context-aware generation

Use `generateWithContext(...)` when you want to separate:
- system instruction
- external context
- end-user message

```kotlin
val system = "You are a helpful assistant."
val context = "Project: Llamatik is a Kotlin-first llama.cpp integration."
val user = "Write a short README intro."

val text = LlamaBridge.generateWithContext(system, context, user)
```

This is particularly useful for RAG-like workflows and structured prompting.

## Streaming instead of waiting

If the response may be long, prefer streaming so the UI can render tokens progressively.
See the dedicated **Streaming** guide for details.

## Cancelling generation

```kotlin
LlamaBridge.nativeCancelGenerate()
```

Call this when:
- the user presses Stop
- the screen is closed
- you no longer need the result

## Resource lifecycle

Do not call `shutdown()` after every generation. In most apps, you load the model once, reuse it across multiple prompts, and release it only when the feature is torn down or the app is terminating.
