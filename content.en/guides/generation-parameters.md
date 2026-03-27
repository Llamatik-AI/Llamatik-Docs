---
title: "Generation parameters"
weight: 4
---

Llamatik exposes a small set of runtime generation parameters through `updateGenerateParams(...)`.

```kotlin
LlamaBridge.updateGenerateParams(
    temperature = 0.7f,
    maxTokens = 256,
    topP = 0.95f,
    topK = 40,
    repeatPenalty = 1.1f,
)
```

## What each parameter does

### Temperature
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
This is often useful for chat, summaries, and structured outputs.

## Tuning advice

- Start from moderate values and test on a fixed prompt set.
- Change one parameter at a time.
- For extraction or JSON, lean toward lower temperature.
- For brainstorming or creative tasks, slightly higher temperature can help.
