---
title: "Migration Guide"
weight: 40
bookFlatSection: false
---

# Migration Guide: 1.x → 2.0

Llamatik 2.0 introduces a new `:sdk` module that extracts all reusable business logic into a clean, UI-agnostic, publishable library. The low-level C++ bridges (`LlamaBridge`, `WhisperBridge`, `StableDiffusionBridge`, `MultimodalBridge`) are **unchanged** — you only need to update Gradle dependency coordinates and import paths.

## Step 1 — Update Gradle dependencies

### Before (1.x)

```kotlin
implementation("com.llamatik:library:<version>")
```

### After (2.0)

```kotlin
// Core inference layer — all JNI bridges (API unchanged)
implementation("com.llamatik:core:<version>")

// Business logic SDK — chat, models, RAG, agents, assistant facade
implementation("com.llamatik:sdk:<version>")

// Agent Platform runtime (optional) — AgentRuntime, permissions, workflows
implementation("com.llamatik:sdk-agent:<version>")
```

## Step 2 — Update package imports

| 1.x Package | 2.0 Package |
|---|---|
| `com.llamatik.library.*` | `com.llamatik.core.*` or `com.llamatik.sdk.*` |
| `com.llamatik.app.feature.chatbot.model.*` | `com.llamatik.sdk.model.*` |
| `com.llamatik.app.feature.chatbot.utils.*` | `com.llamatik.sdk.chat.*` |
| `com.llamatik.app.feature.chatbot.repositories.*` | `com.llamatik.sdk.model.*` or `com.llamatik.sdk.chat.*` |
| `com.llamatik.app.feature.chatbot.download.*` | `com.llamatik.sdk.download.*` |
| `com.llamatik.app.feature.agent.*` | `com.llamatik.sdk.agent.*` |
| `com.llamatik.app.feature.entitlement.*` | `com.llamatik.sdk.entitlement.*` |
| `com.llamatik.app.common.usecases.*` | `com.llamatik.sdk.usecase.*` |
| `com.llamatik.app.platform.ServiceClient` | `com.llamatik.sdk.http.LlamatikHttpClient` |

## Step 3 — Fix breaking changes

### `PromptRenderer` — explicit `Labels` required

The renderer no longer depends on the app's `getCurrentLocalization()`. Pass a `Labels` object:

```kotlin
// 1.x
PromptRenderer.render(system, contexts, messages, template)

// 2.0
val labels = PromptRenderer.Labels(
    system          = getString(R.string.system),
    user            = getString(R.string.user),
    assistant       = getString(R.string.assistant),
    relevantContext = getString(R.string.relevant_context),
)
PromptRenderer.render(system, contexts, messages, template, labels)
```

### `ModelsRepository` — requires `LlamatikFileAccess`

```kotlin
// 1.x
val repo = ModelsRepository(serviceClient)

// 2.0
val repo = ModelsRepository(
    fileAccess = myFileAccess,    // implement LlamatikFileAccess for your platform
    httpClient = LlamatikHttpClient(),
)
```

### `ChatRunner` — `Labels` injected

```kotlin
// 1.x
ChatRunner.stream(session, system, contexts, messages, template, ...) { ... }

// 2.0
val labels = PromptRenderer.Labels(/* localised strings */)
ChatRunner.stream(session, system, contexts, messages, template, labels = labels) { ... }
```

## Type mapping — Chat & Prompts

| 1.x | 2.0 |
|---|---|
| `com.llamatik.app.feature.chatbot.utils.ChatMessage` | `com.llamatik.sdk.chat.ChatMessage` |
| `com.llamatik.app.feature.chatbot.utils.PromptTemplate` | `com.llamatik.sdk.chat.PromptTemplate` |
| `com.llamatik.app.feature.chatbot.utils.Plain` | `com.llamatik.sdk.chat.Plain` |
| `com.llamatik.app.feature.chatbot.utils.Gemma3` | `com.llamatik.sdk.chat.Gemma3` |
| `com.llamatik.app.feature.chatbot.utils.Llama3Instruct` | `com.llamatik.sdk.chat.Llama3Instruct` |
| `com.llamatik.app.feature.chatbot.utils.QwenChat` | `com.llamatik.sdk.chat.QwenChat` |
| `com.llamatik.app.feature.chatbot.utils.PromptRenderer` | `com.llamatik.sdk.chat.PromptRenderer` |
| `com.llamatik.app.feature.chatbot.utils.ChatRunner` | `com.llamatik.sdk.chat.ChatRunner` |
| `com.llamatik.app.feature.chatbot.utils.chunkText()` | `com.llamatik.sdk.rag.chunkText()` |

## Type mapping — Models

| 1.x | 2.0 |
|---|---|
| `com.llamatik.app.feature.chatbot.model.LlamaModel` | `com.llamatik.sdk.model.LlamaModel` |
| `com.llamatik.app.feature.chatbot.model.GenerateSettings` | `com.llamatik.sdk.model.GenerateSettings` |
| `com.llamatik.app.feature.chatbot.model.ModelSource` | `com.llamatik.sdk.model.ModelSource` |
| `com.llamatik.app.feature.chatbot.repositories.ModelsRepository` | `com.llamatik.sdk.model.ModelsRepository` |
| `com.llamatik.app.feature.chatbot.repositories.ChatHistoryRepository` | `com.llamatik.sdk.chat.ChatHistoryRepository` |
| `com.llamatik.app.feature.chatbot.repositories.ChatSession` | `com.llamatik.sdk.chat.ChatSession` |
| `com.llamatik.app.feature.chatbot.usecases.GetModelsUseCase` | `com.llamatik.sdk.model.GetModelsUseCase` |
| `com.llamatik.app.feature.chatbot.usecases.ImportModelUseCase` | `com.llamatik.sdk.model.ImportModelUseCase` |
| `com.llamatik.app.feature.chatbot.download.ModelDownloadOrchestrator` | `com.llamatik.sdk.download.ModelDownloadOrchestrator` |
| `com.llamatik.app.feature.chatbot.download.DownloadEvent` | `com.llamatik.sdk.download.DownloadEvent` |

## Type mapping — Agent

| 1.x | 2.0 |
|---|---|
| `com.llamatik.app.feature.agent.AgentTool` | `com.llamatik.sdk.agent.AgentTool` |
| `com.llamatik.app.feature.agent.AgentToolResult` | `com.llamatik.sdk.agent.AgentToolResult` |
| `com.llamatik.app.feature.agent.AgentExecutor` | `com.llamatik.sdk.agent.AgentExecutor` |
| `com.llamatik.app.feature.agent.ToolRegistry` | `com.llamatik.sdk.agent.ToolRegistry` |
| `com.llamatik.app.feature.agent.ToolCallParser` | `com.llamatik.sdk.agent.ToolCallParser` |
| `com.llamatik.app.feature.agent.AgentFeatureFlags` | `com.llamatik.sdk.agent.AgentFeatureFlags` |
| `com.llamatik.app.feature.agent.AgentActionLogRepository` | `com.llamatik.sdk.agent.AgentActionLogRepository` |
| `com.llamatik.app.feature.agent.ToolPermissionRepository` | `com.llamatik.sdk.agent.ToolPermissionRepository` |

## Type mapping — RAG / Vector Store

| 1.x | 2.0 |
|---|---|
| `com.llamatik.app.feature.chatbot.utils.VectorStoreItem` | `com.llamatik.sdk.rag.VectorStoreItem` |
| `com.llamatik.app.feature.chatbot.utils.VectorStoreData` | `com.llamatik.sdk.rag.VectorStoreData` |
| `com.llamatik.app.feature.chatbot.utils.PersistedRagStore` | `com.llamatik.sdk.rag.PersistedRagStore` |
| `com.llamatik.app.feature.chatbot.utils.retrieveContext()` | `com.llamatik.sdk.rag.retrieveContext()` |
| `com.llamatik.app.feature.chatbot.utils.cosineD()` | `com.llamatik.sdk.rag.cosineD()` |
| `com.llamatik.app.feature.chatbot.utils.mmr()` | `com.llamatik.sdk.rag.mmr()` |

## New in 2.0

Once migrated, explore the new capabilities:

- [SDK Overview](../guides/sdk-overview/) — Chat, models, RAG, and the Assistant facade
- [Agent Framework](../guides/agent-framework/) — `Agent { }` DSL, memory, streaming, workflows
- [Agent Platform](../guides/agent-platform/) — `AgentRuntime`, permissions, built-in tools
