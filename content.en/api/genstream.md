---
title: "GenStream"
weight: 2
---

`GenStream` is the callback interface used by Llamatik's streaming generation APIs.

```kotlin
interface GenStream {
    fun onDelta(text: String)
    fun onComplete()
    fun onError(message: String)
}
```

## When to use it

Use `GenStream` when you want to show output progressively instead of waiting for the full answer.
This is the typical choice for chat UIs, terminals, typing animations, or any screen where responsiveness matters.

## Callback lifecycle

### `onDelta(text: String)`
Called every time the native layer produces a new chunk of text.

Typical usage:

```kotlin
val buffer = StringBuilder()

LlamaBridge.generateStream(
    prompt = "Write a short poem about Kotlin.",
    callback = object : GenStream {
        override fun onDelta(text: String) {
            buffer.append(text)
            println(buffer.toString())
        }

        override fun onComplete() {
            println("Done")
        }

        override fun onError(message: String) {
            println("Error: $message")
        }
    }
)
```

### `onComplete()`
Called once streaming finishes successfully.

Good places to use it:
- hide a loading indicator
- enable the send button again
- persist the final assistant message

### `onError(message: String)`
Called if generation fails or the stream cannot be started.

You should surface this to logs and to the user interface when appropriate.

## Best practices

- Append deltas to a buffer instead of replacing the whole output each time.
- Keep UI updates lightweight, especially on mobile.
- Cancel generation when the screen leaves composition or the user explicitly stops generation.
- Avoid capturing long-lived UI objects in the callback if the stream may outlive the screen.
