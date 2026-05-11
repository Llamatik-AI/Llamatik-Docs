---
title: "MultimodalBridge"
weight: 5
---

`MultimodalBridge` provides on-device vision-language model (VLM) inference.
It combines a language model with a multimodal projector to analyze images and answer questions about them.

## Platform support

- **Android**: supported
- **iOS**: supported
- **JVM / Desktop**: supported
- **WASM**: model paths are stored, but analysis is dispatched to a Web Worker — the worker must be wired up separately

## Public API

```kotlin
expect object MultimodalBridge {
    fun initModel(modelPath: String, mmprojPath: String): Boolean
    fun analyzeImageBytesStream(
        imageBytes: ByteArray,
        prompt: String,
        callback: GenStream,
    )
    fun cancelAnalysis()
    fun release()
}
```

## `initModel(modelPath, mmprojPath)`
Loads both the language model and the multimodal projector (mmproj).
Returns `true` on success.

```kotlin
val modelPath = LlamaBridge.getModelPath("llava-v1.6-mistral-7b.Q4_K_M.gguf")
val mmprojPath = LlamaBridge.getModelPath("llava-v1.6-mistral-7b-mmproj-model-f16.gguf")

val ok = MultimodalBridge.initModel(modelPath, mmprojPath)
check(ok) { "Failed to initialize VLM model" }
```

Both the main model and the mmproj file must come from the same model family and release.

## `analyzeImageBytesStream(imageBytes, prompt, callback)`
Analyzes an image and streams the model's response token by token.
The image can be provided as JPEG, PNG, or BMP bytes — the bridge handles decoding.

This method blocks until the stream completes.
Always call it from a background thread.

```kotlin
val imageBytes: ByteArray = /* JPEG or PNG bytes */

MultimodalBridge.analyzeImageBytesStream(
    imageBytes = imageBytes,
    prompt = "Describe what you see in this image.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

### Parameters

- `imageBytes`: raw image file bytes (JPEG, PNG, or BMP)
- `prompt`: the question or instruction about the image
- `callback`: a [GenStream]({{< relref "genstream" >}}) instance that receives tokens, completion, and error events

### Notes

- Call from a background thread — this call blocks for the duration of the stream.
- Use `cancelAnalysis()` to interrupt a running analysis.

## `cancelAnalysis()`
Cancels a running `analyzeImageBytesStream` call.

```kotlin
MultimodalBridge.cancelAnalysis()
```

Safe to call from any thread.
The in-progress stream will stop and `onComplete` or `onError` will be called to finalize the callback.

## `release()`
Frees the model, the multimodal projector context, and the language model context.

```kotlin
MultimodalBridge.release()
```

Call this when leaving the vision feature or when the app is shutting down.
Do not call while `analyzeImageBytesStream` is still running.
