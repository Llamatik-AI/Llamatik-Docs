---
title: "Stable Diffusion image generation"
weight: 7
---

`StableDiffusionBridge` lets you generate images locally from text prompts.
The bridge returns raw RGBA pixel data so your app can convert it into the most appropriate image type for the current platform.

## Initialize the model

```kotlin
val modelPath = StableDiffusionBridge.getModelPath("dreamshaper.safetensors")
val ok = StableDiffusionBridge.initModel(modelPath, threads = 4)
check(ok)
```

## Generate an image

```kotlin
val rgba = StableDiffusionBridge.txt2img(
    prompt = "A cinematic illustration of a llama explorer",
    negativePrompt = "blurry, low quality, extra limbs",
    width = 512,
    height = 512,
    steps = 20,
    cfgScale = 7.0f,
    seed = 1234L,
)
```

## Understanding the result

The output is an RGBA `ByteArray`.
To display it, convert the bytes into your platform image type using the same width and height you requested.

## Parameter guidance

- `width` and `height`: start with `512x512` unless you have benchmarked larger values
- `steps`: around 20 is a good mobile-friendly default
- `cfgScale`: around 7.0 to 8.0 is a common starting range
- `seed`: use a fixed value for reproducible results

## Performance advice

Image generation is significantly heavier than text generation.
On mobile devices:
- prefer moderate output sizes
- avoid excessive step counts for the first iteration
- reuse one initialized model for many generations

## Cleanup

```kotlin
StableDiffusionBridge.release()
```
