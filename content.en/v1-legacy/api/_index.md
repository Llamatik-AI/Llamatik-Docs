---
title: "API reference"
weight: 30
---

{{< hint warning >}}
**⚠️ Legacy (v1.x)** — You are viewing the old documentation. [View current v2.0 docs](/)
{{< /hint >}}

Llamatik keeps the public API intentionally small, but each bridge covers a different on-device AI task:

- `LlamaBridge` — text generation, embeddings, streaming, JSON output, runtime tuning, model introspection, and KV cache sessions
- `LlamaSession` — isolated concurrent inference session with its own KV cache
- `MultimodalBridge` — on-device vision-language model (VLM) image analysis
- `StableDiffusionBridge` — on-device text-to-image and image-to-image generation
- `WhisperBridge` — on-device speech-to-text from WAV audio
- `GenStream` — callback interface for streaming token output

If you are new to the library, start with **Getting started** first, then come back here for the exact function reference and usage patterns.
