---
title: "Models"
weight: 3
---

Model selection has the biggest impact on speed, memory usage, and output quality.

## LlamaBridge models

For text generation and embeddings, Llamatik works with **GGUF** models.

### Text generation
Choose an instruction-tuned GGUF model when building chat, assistants, extraction, or summarization features.

### Embeddings
Use a model specifically intended for embeddings when calling `initEmbedModel(...)` and `embed(...)`.
Do not assume your generation model is a good embedding model.

## Quantization

GGUF models are often distributed in multiple quantizations.
The tradeoff is straightforward:

- smaller quantizations: lower memory use, faster inference, lower quality
- larger quantizations: higher memory use, slower inference, often better quality

A good development strategy is:

1. start with a small quantized model to validate your integration
2. move to a larger target model once everything is working

## Stable Diffusion models

For `StableDiffusionBridge`, use a model compatible with the native backend used by the library.
Since image generation is heavier than text generation, start with conservative image sizes and settings while validating performance.

## Whisper models

For `WhisperBridge`, choose a model size that matches your latency and accuracy goals.
Smaller models are faster and lighter; larger models are usually more accurate.

## Shipping strategy

Models can be large, so most apps choose one of these approaches:

- bundle a small default model
- download models after installation
- let advanced users choose which models to download

## Practical advice

- Keep one model per task at first: one text model, one embedding model, one Whisper model, one image model.
- Reuse initialized models rather than loading them repeatedly.
- Test on real target hardware, especially for mobile image generation.
