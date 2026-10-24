---
id: glossary
title: Glossary
doc_type: glossary
status: current
app_version: 1.0.20
last_reviewed: 2026-10-24
audience: [everyone, ai-agents]
related: [architecture, agents, technical-design]
sources:
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/data/
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/agent/
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/tools/
---

# Glossary

Terms are alphabetical. `Code` names a type or constant. The code column says where to look.

| Term | Definition | Code |
|------|------------|------|
| **Accelerator** | Hardware backend for inference: CPU, GPU, NPU, or TPU (on Pixel the NPU is relabelled TPU). | `Accelerator`, `BackendSpec.accelerators` |
| **AgentRuntimeExecutor** | Runs agent turns (ReAct loop) on an initialized model and streams `AgentEvent`s. | `agent/AgentRuntimeExecutor.kt` |
| **Agent Skills** | The task (`llm_agent_chat`) where the LLM can use skills, MCP tools, and intents. | `customtasks/agentchat/` |
| **AICore** | Android system service that hosts on-device foundation models. Reached through ML Kit GenAI Prompt. | `runtime/aicore/` |
| **Allowlist (model)** | Versioned JSON catalog of models the app offers, fetched from GitHub. | `data/ModelAllowlist.kt`, `model_allowlists/` |
| **Allowlist (skill)** | Remote list of featured skills. URL empty in this build. | `data/SkillAllowlist.kt`, `SKILL_ALLOWLIST_URL` |
| **appassets origin** | `https://appassets.androidplatform.net`. The virtual origin that serves bundled and imported skill files to the WebView. | `LOCAL_URL_BASE`, `GalleryWebView` |
| **BackendSpec** | A model's runtime type, AICore stage and preference, and accelerators. | `data/BackendSpec.kt` |
| **Built-in task** | A task shipped with the app, with an id in `BuiltInTaskId`. | `data/Tasks.kt` |
| **Capability** | An optional model feature (`llm_thinking`, `speculative_decoding`), gated per task by `capabilityToTaskTypes`. | `ModelCapability` |
| **Category** | A home-screen tab grouping tasks (`llm`, `classical_ml`, `experimental`). | `data/Categories.kt` |
| **Context compaction** | Summarizing a long conversation and resetting the KV cache. Disabled in this build. | `SummarizationContextCompactor` |
| **CustomTask** | The plug-in interface every feature implements. | `customtasks/common/CustomTask.kt` |
| **EmbeddingGemma 2** | Multimodal embedding model behind Instant Media Search and Video Moment Finder. | `services/semanticretrieval/` |
| **Extra data files** | Optional components downloaded with a model (e.g. `samples.zip`). | `ModelDataFile` |
| **FunctionGemma** | 270M Gemma fine-tunes for function calling (Mobile Actions, Tiny Garden). | allowlist `MobileActions-270M`, `TinyGarden-270M` |
| **Gated model** | A Hugging Face model that needs license acceptance and an auth token. | `ModelAccessibility.GATED` |
| **Imported model** | A model file the user adds manually, stored under `__imports/`. | `ImportedModel` (proto) |
| **Intent (agent)** | A native Android action the model can run through `run_intent`. | `intents/IntentHandler.kt` |
| **LiteRT** | Google's on-device ML runtime (formerly TensorFlow Lite). | — |
| **LiteRT-LM** | LiteRT's LLM engine (`Engine`, `Conversation`, tools, constrained decoding). `.litertlm` files. | `LlmChatModelHelper` |
| **LlmModelHelper** | The runtime abstraction for LLMs. | `runtime/LlmModelHelper.kt` |
| **LlmSessionManager** | Owns conversation sessions, history, inference calls, and compaction. | `agent/sessions/` |
| **MCP** | Model Context Protocol. Remote tool servers the agent can call. | `McpManagerViewModel`, `RunMcpTool` |
| **Model** | A runnable model entry (from the allowlist, an import, or inline in a task). | `data/Model.kt` |
| **ModelDownloader** | Per-runtime strategy for status, download, cancel, and delete. | `data/ModelDownloader.kt` |
| **normalizedName** | Model name with non-alphanumerics replaced by `_`. Used for storage paths. | `Model.normalizedName` |
| **Runtime helper** | The `LlmModelHelper` picked for a model (`Model.runtimeHelper`). | `runtime/ModelHelperExt.kt` |
| **RuntimeType** | `litert_lm`, `aicore`, `unknown`, with behaviour flags (download via repo, auto-download, deletable). | `data/ModelEnums.kt` |
| **Session** | A conversation history, identified by a UUID and independent of the model. | `generateSessionId()` |
| **Skill** | An instruction pack (`SKILL.md`) plus optional JS scripts and assets. | `skills/`, `skill.proto` |
| **SoC model files** | Per-chipset NPU builds of a model, chosen by `Build.SOC_MODEL`. | `socToModelFiles` |
| **Supplementary model** | An allowlist model not bound to a task (`modelType != task`). | `Model.isSupplementary` |
| **Task** | A home-screen feature: metadata, models, and capability flags. | `data/Tasks.kt` |
| **Tool** | A Kotlin method exposed to the model with `@Tool`. | `tools/ToolDefinition.kt` |
| **Tool action** | An event from a running tool to the UI (run JS, ask a secret, request permission, MCP consent, progress). | `tools/ToolAction.kt` |
| **`.gallerytmp`** | Extension of partially downloaded files, used to resume downloads. | `TMP_FILE_EXT` |
