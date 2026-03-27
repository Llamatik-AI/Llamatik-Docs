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
