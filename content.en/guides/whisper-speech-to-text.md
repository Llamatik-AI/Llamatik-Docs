---
title: "Whisper speech-to-text"
weight: 8
---

`WhisperBridge` is the speech-to-text entry point in Llamatik.
The library currently exposes a WAV-based transcription API.

## Initialize the model

```kotlin
val modelPath = WhisperBridge.getModelPath("ggml-base.en.bin")
val ok = WhisperBridge.initModel(modelPath)
check(ok)
```

## Transcribe a file

```kotlin
val text = WhisperBridge.transcribeWav(
    wavPath = "/path/to/audio.wav",
    language = "en"
)
println(text)
```

## Language hint

The `language` parameter is optional, but supplying it can improve reliability when you already know the input language.

## Initial prompt

The `initialPrompt` parameter primes the model before transcription begins.
Use it to bias output toward specific vocabulary, domain terms, or formatting conventions.

```kotlin
val text = WhisperBridge.transcribeWav(
    wavPath = audioPath,
    language = "en",
    initialPrompt = "The following is a developer podcast about Kotlin Multiplatform."
)
```

The model uses the prompt as prior context — it does not transcribe it literally.
This is useful when your audio contains technical terms or proper nouns that the model might otherwise mis-transcribe.

## Recommended workflow

1. record or obtain audio
2. convert it to WAV if needed
3. initialize the model once
4. call `transcribeWav(...)` for each file
5. release the model when done

## Cleanup

```kotlin
WhisperBridge.release()
```

## Practical notes

- For best results, keep input audio clear and reasonably clean.
- Reuse the initialized model if you transcribe multiple files.
- WASM support is currently not available for WhisperBridge.
