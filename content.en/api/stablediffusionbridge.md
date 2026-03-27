---
title: "StableDiffusionBridge"
weight: 3
---

`StableDiffusionBridge` provides on-device text-to-image generation.
It is intended for apps that want local image generation without sending prompts or generated images to external servers.

## Platform support

Current support in the library code:

- **Android**: supported
- **iOS**: supported
- **JVM / Desktop**: supported
- **WASM**: currently not implemented

## Public API

```kotlin
expect object StableDiffusionBridge {
    fun getModelPath(modelFileName: String): String
    fun initModel(modelPath: String, threads: Int = -1): Boolean
    fun txt2img(
        prompt: String,
        negativePrompt: String? = null,
        width: Int = 512,
        height: Int = 512,
        steps: Int = 20,
        cfgScale: Float = 7.0f,
        seed: Long = -1L,
    ): ByteArray
    fun release()
}
```

## `getModelPath(modelFileName)`
Returns a usable path for the model file on the current platform.

```kotlin
val modelPath = StableDiffusionBridge.getModelPath("dreamshaper.safetensors")
```

## `initModel(modelPath, threads)`
Loads the Stable Diffusion model and creates the native context.

```kotlin
val ok = StableDiffusionBridge.initModel(modelPath, threads = 4)
check(ok) { "Failed to initialize Stable Diffusion model" }
```

### Parameters

- `modelPath`: absolute or platform-resolved path to the model file
- `threads`: CPU threads to use. Pass `-1` to let the backend decide

You normally call this once and reuse the loaded model for multiple generations.

## `txt2img(...)`
Generates one image and returns the raw result as an **RGBA byte array**.

```kotlin
val rgba = StableDiffusionBridge.txt2img(
    prompt = "A cinematic photo of a llama astronaut",
    negativePrompt = "blurry, low quality, distorted",
    width = 512,
    height = 512,
    steps = 20,
    cfgScale = 7.5f,
    seed = 42L,
)
```

### Parameters

- `prompt`: what you want to generate
- `negativePrompt`: what you want the model to avoid
- `width` / `height`: output size in pixels
- `steps`: number of diffusion steps. Higher usually means slower but can improve quality
- `cfgScale`: how strongly the model should follow the prompt
- `seed`: controls reproducibility. Use a fixed value for repeatable output, or `-1L` for non-deterministic behavior

### Return value

The function returns an RGBA `ByteArray` of size `width * height * 4`.
If generation fails, the returned array may be empty.

## Converting the RGBA result

The bridge returns raw pixels so you can decide how to display or save them on each platform.

Typical workflow:
- Android: convert the bytes into a `Bitmap`
- iOS: convert into `UIImage` / CoreGraphics image data
- Desktop: convert into `BufferedImage`

Pseudo-code idea:

```kotlin
val rgba = StableDiffusionBridge.txt2img(...)
check(rgba.isNotEmpty())
// Convert width/height/RGBA bytes into a platform image type
```

## `release()`
Frees the native Stable Diffusion context.

```kotlin
StableDiffusionBridge.release()
```

Call this when you are done with image generation or when the feature is being torn down.

## Practical recommendations

- Start with `512x512`, `steps = 20`, `cfgScale = 7.0f`.
- Use a negative prompt for better consistency when quality matters.
- Reuse one initialized model for multiple generations instead of loading it every time.
- Expect significant CPU and memory use on mobile devices.
