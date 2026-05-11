---
title: "Generation parameters"
weight: 4
---

Llamatik exposes runtime generation parameters through `updateGenerateParams(...)`.

```kotlin
LlamaBridge.updateGenerateParams(
    temperature = 0.7f,
    maxTokens = 256,
    topP = 0.95f,
    topK = 40,
    repeatPenalty = 1.1f,
    contextLength = 4096,
    numThreads = 4,
    useMmap = true,
    flashAttention = false,
    batchSize = 512,
)
```

## Sampling parameters

### temperature
Controls randomness.

- lower values: more deterministic
- higher values: more varied and creative

### maxTokens
Sets the maximum number of tokens the model may generate.
Use this to control response length and latency.

### topP
Nucleus sampling threshold.
The model samples from the smallest set of likely tokens whose cumulative probability reaches `topP`.

### topK
Limits sampling to the `K` most likely next tokens.

### repeatPenalty
Discourages repeated phrases and loops.
Useful for chat, summaries, and structured outputs.

## Backend and memory parameters

### contextLength
The model's context window in tokens.
Larger values allow longer conversations but require more memory.
Must not exceed what the loaded model was compiled to support.

### numThreads
Number of CPU threads used during inference.
Pass `-1` to let the platform choose a sensible default.
On mobile, matching the number of performance cores is a good starting point.

### useMmap
Whether to use memory-mapped I/O for the model weights.
Enabled by default on most platforms.
Turn it off if you encounter issues with certain file systems or need full memory ownership.

### flashAttention
Enables Flash Attention when supported by the backend.
Can reduce memory usage and improve throughput on large context windows.
Not supported on all platforms — verify with the underlying llama.cpp version in use.

### batchSize
The batch size used during prompt processing (the prefill phase).
Larger values can improve throughput on long prompts at the cost of memory.
A value of `512` works well for most use cases.

## Tuning advice

- Start from moderate values and test on a fixed prompt set.
- Change one parameter at a time.
- For extraction or JSON, lean toward lower temperature.
- For brainstorming or creative tasks, slightly higher temperature can help.
- For long chat history, increase `contextLength` and consider enabling `flashAttention`.
- If memory is tight on mobile, reduce `batchSize` and `contextLength`.
