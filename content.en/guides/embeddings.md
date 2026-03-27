---
title: "Embeddings"
weight: 1
---

Embeddings convert text into vectors that preserve semantic meaning.
They are a foundation for search, retrieval, similarity ranking, clustering, and RAG pipelines.

## Initialization

```kotlin
val ok = LlamaBridge.initEmbedModel(modelPath)
check(ok)
```

Use an embedding model here, not your general chat model.

## Generating vectors

```kotlin
val v1 = LlamaBridge.embed("What is Llamatik?")
val v2 = LlamaBridge.embed("Kotlin llama.cpp bindings")
```

The returned `FloatArray` can be stored directly or passed into your vector search library.

## Similarity example

```kotlin
fun cosine(a: FloatArray, b: FloatArray): Float {
    require(a.size == b.size)
    var dot = 0.0
    var na = 0.0
    var nb = 0.0
    for (i in a.indices) {
        val x = a[i].toDouble()
        val y = b[i].toDouble()
        dot += x * y
        na += x * x
        nb += y * y
    }
    return (dot / (kotlin.math.sqrt(na) * kotlin.math.sqrt(nb))).toFloat()
}
```

## Best practices

- Use one embedding model consistently across one index.
- Cache document embeddings instead of recomputing them.
- Keep the raw source text alongside embeddings so you can debug retrieval quality.
- On WASM, embeddings are currently unavailable.
