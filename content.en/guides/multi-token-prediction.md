---
title: "Multi-Token Prediction (MTP)"
weight: 11
---

Multi-Token Prediction (MTP) is a speculative decoding technique where a lightweight draft head — embedded in the same GGUF file as the main model — predicts several tokens ahead of the trunk.
The trunk model then verifies all draft tokens in one batched forward pass and accepts the ones that match.

The result is faster generation with no change in output quality.

## How it works

Without MTP each trunk decode step produces one token.
With MTP enabled:

1. The trunk samples token T₀ normally and emits it.
2. The MTP head receives the trunk's pre-norm hidden state and T₀, and drafts up to `draftLen` additional tokens.
3. The trunk verifies all draft tokens in a single batched forward pass.
4. Accepted tokens are emitted immediately. On the first mismatch, the trunk's corrected token is used instead.
5. When all drafts are accepted, one bonus token is sampled from the final trunk logits at no extra cost.

This gives between 1.5× and 2.5× throughput improvement on CPU for supported models.

## Supported models

MTP layers must be present in the GGUF file. Models known to include them:

- **Qwen3.5** and **Qwen3.5-MoE** (all sizes)
- **GLM-4** family

`initMtp()` returns `false` if the loaded model has no MTP layers, so it is safe to call unconditionally.

## Enable MTP

Call `initMtp()` after `initGenerateModel()`. Pass the same model path — the library loads only the MTP head layers from it.

```kotlin
val modelPath = LlamaBridge.getModelPath("Qwen3.5-1.7B-Instruct-Q4_K_M.gguf")

LlamaBridge.initGenerateModel(modelPath)

val ok = LlamaBridge.initMtp(modelPath, draftLen = 3)
if (!ok) {
    // Model has no MTP layers — generation continues at normal speed
}
```

The `draftLen` parameter sets the maximum number of speculative tokens per step.
Values between 1 and 8 are recommended. The default is 3.

Higher values can increase throughput further on long, predictable outputs, but every misprediction wastes the work done for all drafts in that step.
Start with 3 and tune from there.

## Use the API as normal

Once MTP is enabled, no changes to generation calls are required. The speculative loop runs transparently inside the bridge.

```kotlin
LlamaBridge.generateStream(
    prompt = "Explain the Kotlin coroutine lifecycle.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(msg: String) = println("Error: $msg")
    }
)
```

## Disable MTP at runtime

MTP resources can be released without unloading the main model:

```kotlin
LlamaBridge.shutdownMtp()
```

Generation continues normally using the trunk only.
Call `initMtp()` again at any time to re-enable it.

## Lifecycle

- `initMtp()` must be called **after** `initGenerateModel()`.
- `shutdown()` releases MTP resources automatically — calling `shutdownMtp()` first is not required.
- Reloading the generation model with `initGenerateModel()` does not automatically re-enable MTP; call `initMtp()` again after.

## Scope

MTP is active for all streaming generation methods:

- `generateStream()`
- `generateStreamWithContext()`
- `generateWithContextStream()`

Non-streaming methods (`generate()`, `generateWithContext()`) and JSON-constrained methods run the standard single-token decode loop regardless of whether MTP is enabled.

## Platform notes

| Platform | MTP support |
|---|---|
| Android | Yes |
| iOS | Yes |
| JVM / Desktop | Yes |
| WASM | No — `initMtp()` always returns `false` |
