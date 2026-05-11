---
title: "LlamaBridge"
weight: 1
---

`LlamaBridge` is the main multiplatform entry point for GGUF-based language models in Llamatik.
It exposes a compact API for:

- text generation
- retrieval-friendly embeddings
- streaming tokens
- JSON and JSON Schema constrained output
- generation parameter tuning
- KV cache reuse and session persistence
- model introspection (finetune type, chat template)
- concurrent independent sessions via `LlamaSession`

Under the hood, the actual implementation is platform-specific, but the public API stays the same.

## Platform support

Current support in the library code:

- **Android**: supported
- **iOS**: supported
- **JVM / Desktop**: supported
- **WASM**: text generation is supported, but embeddings and KV session persistence are currently not available, and synchronous `generateContinue()` is not supported in worker-only mode

## Model path helper

```kotlin
fun getModelPath(modelFileName: String): String
```

This returns a platform-usable path for a model file.

Typical usage:

```kotlin
val modelPath = LlamaBridge.getModelPath("qwen2.5-0.5b-instruct-q4_k_m.gguf")
```

Use this when your app stores or ships models differently across platforms and you want one common entry point before initialization.

## Embeddings API

```kotlin
fun initEmbedModel(modelPath: String): Boolean
fun embed(input: String): FloatArray
```

### `initEmbedModel(modelPath)`
Loads a model for embeddings.
Returns `true` on success.

```kotlin
val ok = LlamaBridge.initEmbedModel(modelPath)
check(ok) { "Failed to initialize embedding model" }
```

### `embed(input)`
Computes a vector representation of the input text.
This is useful for:

- semantic search
- document retrieval
- clustering
- reranking pipelines
- PDF RAG and other retrieval workflows

```kotlin
val vector = LlamaBridge.embed("How do coroutines work?")
println("Embedding size = ${vector.size}")
```

Important notes:
- Use the same embedding model for all vectors inside one index.
- Initialize the embedding model before calling `embed`.
- On WASM, embeddings are currently not implemented.

## Generation model initialization

```kotlin
fun initGenerateModel(modelPath: String): Boolean
```

Loads a generation model and prepares it for inference.

```kotlin
val ok = LlamaBridge.initGenerateModel(modelPath)
check(ok) { "Failed to initialize generation model" }
```

Call this before any generation method.

## One-shot generation

```kotlin
fun generate(prompt: String): String
```

Generates a full response from a single prompt.
This is the simplest API and a good default when each request is independent.

```kotlin
val answer = LlamaBridge.generate("Explain Kotlin Multiplatform in one paragraph.")
println(answer)
```

`generate(...)` starts a fresh generation flow. If you are building a chat interface and want continuity across turns, prefer the session APIs described below.

## Context-aware generation

```kotlin
fun generateWithContext(systemPrompt: String, contextBlock: String, userPrompt: String): String
```

Use this when you want clearer structure between instructions, external context, and user input.

```kotlin
val result = LlamaBridge.generateWithContext(
    systemPrompt = "You are a concise technical assistant.",
    contextBlock = "Project: Llamatik is a Kotlin Multiplatform library for on-device AI.",
    userPrompt = "Write a short summary for the README."
)
```

This is useful for:
- RAG results
- chat prompts with a fixed system role
- structured prompting where you want deterministic sections

## JSON generation

```kotlin
fun generateJson(prompt: String, jsonSchema: String? = null): String
fun generateJsonWithContext(
    systemPrompt: String,
    contextBlock: String,
    userPrompt: String,
    jsonSchema: String? = null
): String
```

These methods instruct the model to return JSON output. If you provide a JSON Schema, generation is additionally constrained to that schema.

```kotlin
val schema = """
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "title": { "type": "string" },
    "priority": { "type": "integer" }
  },
  "required": ["title", "priority"]
}
""".trimIndent()

val json = LlamaBridge.generateJson(
    prompt = "Return one task object.",
    jsonSchema = schema
)
```

This is especially useful when parsing model output into Kotlin data classes.

## Streaming APIs

```kotlin
fun generateStream(prompt: String, callback: GenStream)
fun generateStreamWithContext(systemPrompt: String, contextBlock: String, userPrompt: String, callback: GenStream)
fun generateJsonStream(prompt: String, jsonSchema: String? = null, callback: GenStream)
fun generateJsonStreamWithContext(
    systemPrompt: String,
    contextBlock: String,
    userPrompt: String,
    jsonSchema: String? = null,
    callback: GenStream
)
```

These are the streaming equivalents of the one-shot APIs above.
They are the best choice for chat interfaces because the UI can render tokens incrementally.

```kotlin
LlamaBridge.generateStream(
    prompt = "Write a haiku about local AI.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Lambda-friendly streaming helper

```kotlin
fun generateWithContextStream(
    system: String,
    context: String,
    user: String,
    onDelta: (String) -> Unit,
    onDone: () -> Unit,
    onError: (String) -> Unit
)
```

This is a convenience wrapper around the callback-based context streaming API when you prefer lambdas.

```kotlin
LlamaBridge.generateWithContextStream(
    system = "You are helpful.",
    context = "Product: Llamatik",
    user = "Write a short tagline.",
    onDelta = { print(it) },
    onDone = { println("\nDone") },
    onError = { println("Error: $it") }
)
```

## KV cache and sessions

```kotlin
fun sessionReset(): Boolean
fun sessionSave(path: String): Boolean
fun sessionLoad(path: String): Boolean
fun generateContinue(prompt: String): String
```

These methods are designed for multi-turn interactions.

### `generateContinue(prompt)`
Continues generation using the current KV cache instead of starting from scratch.

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val first = LlamaBridge.generate("Explain Kotlin coroutines")
val second = LlamaBridge.generateContinue("Now show a short example")
```

### `sessionSave(path)` and `sessionLoad(path)`
Persist the current KV/session state to disk and restore it later.

```kotlin
LlamaBridge.sessionSave(sessionPath)

LlamaBridge.initGenerateModel(modelPath)
LlamaBridge.sessionLoad(sessionPath)

val resumed = LlamaBridge.generateContinue("Continue the explanation")
```

### `sessionReset()`
Clears the current session while keeping the model loaded.
This is useful when you want to start a new conversation without paying the full model load cost again.

Important notes:
- Session files are tied to the same model and runtime assumptions.
- On WASM, session persistence is currently not implemented.
- If no active session exists yet, `generateContinue()` behaves like a fresh generation.

## Model introspection

```kotlin
fun getModelFinetuneType(): String?
fun getModelChatTemplate(): String?
fun applyChatTemplate(messages: List<Pair<String, String>>, addAssistantPrefix: Boolean): String?
```

### `getModelFinetuneType()`
Returns the `general.finetune` key embedded in the GGUF metadata, such as `"instruct"` or `"chat"`, or `null` if the field is not present.
Useful when deciding whether a model needs a chat template applied.

```kotlin
val type = LlamaBridge.getModelFinetuneType()
println(type) // e.g. "instruct"
```

### `getModelChatTemplate()`
Returns the Jinja-style chat template embedded in the GGUF file, or `null` if none is present.
The template describes how messages should be formatted into a prompt string for that model.

```kotlin
val template = LlamaBridge.getModelChatTemplate()
```

### `applyChatTemplate(messages, addAssistantPrefix)`
Formats a list of `(role, content)` message pairs using the model's embedded chat template and returns the final prompt string.
Pass `addAssistantPrefix = true` to append the assistant turn prefix (e.g. `<|assistant|>`) so the model knows to continue from that role.

```kotlin
val messages = listOf(
    "system" to "You are a concise assistant.",
    "user" to "What is KMP?"
)
val prompt = LlamaBridge.applyChatTemplate(messages, addAssistantPrefix = true)
if (prompt != null) {
    LlamaBridge.generateStream(prompt, callback)
}
```

## Concurrent sessions

```kotlin
fun createSession(): LlamaSession?
```

### `createSession()`
Creates an independent `LlamaSession` with its own KV cache and cancel flag.
Returns `null` if no generation model is loaded.
Multiple sessions can stream concurrently on separate threads.

```kotlin
val sessionA = LlamaBridge.createSession() ?: error("model not loaded")
val sessionB = LlamaBridge.createSession() ?: error("model not loaded")

// Stream concurrently
thread { sessionA.stream("Describe the moon", callback) }
thread { sessionB.stream("Describe the sun", callback) }
```

See [LlamaSession]({{< relref "llamasession" >}}) for the full session API.

## Runtime controls

```kotlin
fun nativeCancelGenerate()
fun updateGenerateParams(
    temperature: Float,
    maxTokens: Int,
    topP: Float,
    topK: Int,
    repeatPenalty: Float,
    contextLength: Int,
    numThreads: Int,
    useMmap: Boolean,
    flashAttention: Boolean,
    batchSize: Int,
)
fun shutdown()
```

### `nativeCancelGenerate()`
Requests cancellation of the current generation.
Useful when the user taps Stop or leaves the screen.

### `updateGenerateParams(...)`
Updates the runtime generation parameters used by the native backend.

```kotlin
LlamaBridge.updateGenerateParams(
    temperature = 0.7f,
    maxTokens = 256,
    topP = 0.95f,
    topK = 40,
    repeatPenalty = 1.1f,
    contextLength = 4096,
    numThreads = 4,
    useMmap = true,
    flashAttention = false,
    batchSize = 512,
)
```

### Parameters

- `temperature`: controls randomness. Lower values give more deterministic output.
- `maxTokens`: maximum number of tokens to generate per call.
- `topP`: nucleus sampling probability mass. Typically 0.9–0.95.
- `topK`: restricts sampling to the top K tokens at each step.
- `repeatPenalty`: penalizes repeated tokens. 1.0 means no penalty.
- `contextLength`: the model's context window size in tokens.
- `numThreads`: number of CPU threads for inference. Pass `-1` to use the platform default.
- `useMmap`: whether to use memory-mapped file I/O for the model weights.
- `flashAttention`: enables Flash Attention when supported by the backend.
- `batchSize`: prompt processing batch size. Larger values can improve throughput on long prompts.

### `shutdown()`
Releases native resources when you are done with the bridge.
In long-lived apps this is usually called when the feature or process is shutting down, not after every request.
