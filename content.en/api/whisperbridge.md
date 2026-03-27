---
title: "WhisperBridge"
weight: 4
---

`WhisperBridge` provides on-device speech-to-text using Whisper models.
Its API is intentionally small: initialize a model, transcribe a WAV file, and release resources when done.

## Platform support

Current support in the library code:

- **Android**: supported
- **iOS**: supported
- **JVM / Desktop**: supported
- **WASM**: currently not implemented

## Public API

```kotlin
expect object WhisperBridge {
    fun getModelPath(modelFileName: String): String
    fun initModel(modelPath: String): Boolean
    fun transcribeWav(wavPath: String, language: String? = null): String
    fun release()
}
```

## `getModelPath(modelFileName)`
Returns a platform-usable path for the Whisper model file.

```kotlin
val modelPath = WhisperBridge.getModelPath("ggml-base.en.bin")
```

## `initModel(modelPath)`
Loads the Whisper model and prepares the transcription context.

```kotlin
val ok = WhisperBridge.initModel(modelPath)
check(ok) { "Failed to initialize Whisper model" }
```

Call this once before transcription.

## `transcribeWav(wavPath, language)`
Transcribes a WAV audio file into text.

```kotlin
val text = WhisperBridge.transcribeWav(
    wavPath = "/path/to/sample.wav",
    language = "en"
)
println(text)
```

### Parameters

- `wavPath`: path to the WAV file to transcribe
- `language`: optional language hint such as `"en"`, `"es"`, or `"fr"`

If you already know the language, providing it can improve predictability and reduce ambiguity.

### Return value

Returns the transcription as a `String`.
If transcription fails, the result may be empty.

## Supported input expectations

The bridge API is WAV-oriented, so the simplest and most reliable flow is:

1. record or load audio
2. convert it to WAV if necessary
3. call `transcribeWav(...)`

If your app records audio in another format, perform a conversion step before calling the bridge.

## `release()`
Releases native Whisper resources.

```kotlin
WhisperBridge.release()
```

## Practical recommendations

- Keep audio clean and reasonably short when testing.
- Normalize your pipeline around WAV files for fewer surprises.
- Reuse the initialized model when transcribing multiple files.
- Release resources when leaving the speech feature or shutting the app down.
