---
title: "Installation"
weight: 1
---

## v2.0 modules

Llamatik 2.0 ships three independent artifacts. Add whichever you need:

| Artifact | Description |
|---|---|
| `com.llamatik:core` | JNI bridges only: `LlamaBridge`, `WhisperBridge`, `StableDiffusionBridge`, `MultimodalBridge` |
| `com.llamatik:sdk` | Business logic: chat, models, download, RAG, Agent Framework DSL, `Assistant` facade |
| `com.llamatik:sdk-agent` | Agent Platform runtime: `AgentRuntime`, permissions, profiles, audit, tools (depends on `:sdk`) |

```kotlin
dependencies {
    // Core inference layer (required)
    implementation("com.llamatik:core:<version>")

    // SDK — business logic, chat, RAG, Agent Framework (recommended)
    implementation("com.llamatik:sdk:<version>")

    // Agent Platform runtime (optional)
    implementation("com.llamatik:sdk-agent:<version>")
}
```

> **Upgrading from 1.x?** See the [Migration Guide](../../migration/).

## Kotlin Multiplatform

In a KMP project, add the dependencies to the source sets that need them:

```kotlin
kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("com.llamatik:core:<version>")
                implementation("com.llamatik:sdk:<version>")
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
