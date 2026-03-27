---
title: "Installation"
weight: 1
---

## Gradle dependency

Add the library to the module where you want to use Llamatik.

```kotlin
dependencies {
    implementation("com.llamatik:library:0.x.y")
}
```

## Kotlin Multiplatform

In a KMP project, add the dependency to the source sets that need it.

```kotlin
kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("com.llamatik:library:0.x.y")
            }
        }
    }
}
```

## Platform notes

### Android

- **Min SDK:** 26
- The library loads native code at runtime.
- If you package models in assets, make sure your app can resolve or copy them to a readable file path before initialization.

### iOS

- **Minimum deployment target:** 16.6
- The library is consumed through Kotlin Multiplatform / Kotlin Native integration.
- Model files need to be bundled or downloaded in a way your app can resolve to an actual file path.

### JVM / Desktop

- Use a recent JDK. The repository is configured around toolchain 21.
- Native libraries are loaded from packaged resources.
- Model paths are usually absolute disk paths.

### WASM

- Text generation is available through the WASM implementation.
- Embeddings, Stable Diffusion, Whisper, and KV session persistence are currently not available.
- If you rely on streaming or worker-only execution, prefer the streaming APIs.

## What to install beyond the dependency

Llamatik gives you the native bridge. You still need compatible model files:

- `LlamaBridge`: GGUF text models, and GGUF embedding models for `initEmbedModel`
- `StableDiffusionBridge`: a compatible Stable Diffusion model file
- `WhisperBridge`: a compatible Whisper model file

The next page explains how to think about model selection.
