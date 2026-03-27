---
title: "Schema-constrained JSON"
weight: 5
---

Llamatik can ask the model to return JSON and optionally constrain it to a JSON Schema.
This is one of the most useful features when you want AI output to plug into typed application logic.

## Basic JSON generation

```kotlin
val json = LlamaBridge.generateJson(
    prompt = "Return a JSON object with fields: name (string), year (int)."
)
println(json)
```

## JSON Schema constrained generation

```kotlin
val schema = """
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "name": { "type": "string" },
    "year": { "type": "integer" }
  },
  "required": ["name", "year"]
}
""".trimIndent()

val json = LlamaBridge.generateJson(
    prompt = "Generate an example object.",
    jsonSchema = schema
)
```

## With context

```kotlin
val json = LlamaBridge.generateJsonWithContext(
    systemPrompt = "You output only JSON.",
    contextBlock = "The product is Llamatik.",
    userPrompt = "Return product metadata.",
    jsonSchema = schema
)
```

## Streaming JSON

```kotlin
LlamaBridge.generateJsonStream(
    prompt = "Return JSON for a grocery list with 3 items.",
    jsonSchema = schema,
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Why this matters

Schema-constrained output reduces brittle post-processing and makes it easier to parse results with Kotlin serialization libraries.
It is especially useful for:
- forms and extraction
- tool calling style outputs
- strongly typed app workflows
