---
id: requirements
title: Requirements — Goals, Scope, Functional and Non-Functional
doc_type: requirements
status: mixed
app_version: 1.0.20
last_reviewed: 2026-10-24
audience: [product, engineers, qa, reviewers]
related: [functional-design, security-privacy, testing]
sources:
  - README.md
  - Android/src/app/build.gradle.kts
  - Android/src/app/src/main/AndroidManifest.xml
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/data/Consts.kt
  - model_allowlists/1_0_20.json
---

# Requirements

This repo has no formal requirements document. The requirements below were **reconstructed** from the README and
from behaviour implemented in code **[inferred]**. Each requirement has a stable id so tests and issues can refer to it.
The **Evidence** column points to the implementation.

## 1. Goals

| Id | Goal |
|----|------|
| G-1 | Show state-of-the-art open models running **fully on-device** on Android. |
| G-2 | Let developers and enthusiasts evaluate models on their own hardware (quality, latency, memory). |
| G-3 | Showcase agentic patterns on-device: skills, function calling, MCP, device intents. |
| G-4 | Act as a reference implementation of LiteRT-LM, AICore, and MediaPipe integration. |

## 2. Non-goals

| Id | Non-goal |
|----|----------|
| NG-1 | Cloud inference or server-side processing of user content. |
| NG-2 | Being a general-purpose production assistant. The app is labelled an experimental Beta. |
| NG-3 | Accepting external code contributions (for now; see `CONTRIBUTING.md`). |

## 3. Functional requirements

| Id | Requirement | Evidence |
|----|-------------|----------|
| FR-01 | Show a home screen of tasks grouped by category, built from the registered tasks. | `HomeScreen`, `Set<CustomTask>` |
| FR-02 | Load the model catalog for the running app version from a remote allowlist, with a local cache fallback. | `loadModelAllowlist()` |
| FR-03 | Hide models the device can't run (disabled, unsupported AICore device, NPU-only on an unsupported SoC). | `loadModelAllowlist()` filters |
| FR-04 | Download models in the background with progress, resume after interruption, and allow cancel and delete. | `DownloadWorker`, `DownloadRepository` |
| FR-05 | Support gated Hugging Face models through OAuth sign-in. | `ModelManagerViewModel.handleAuthResult` |
| FR-06 | Let users import their own `.litertlm`/`.task` models and test them in chat. | `ModelImportDialog`, `llm_test` |
| FR-07 | Multi-turn chat with streaming, optional thinking output, image and audio input when the model supports them. | `LlmChatScreen`, `LlmChatModelHelper` |
| FR-08 | Ask Image: up to 10 images per session (1 on AICore). | `MAX_IMAGE_COUNT`, `MAX_IMAGE_COUNT_AI_CORE` |
| FR-09 | Audio Scribe: one clip of up to 30 s at 16 kHz. | `MAX_AUDIO_CLIP_*`, `SAMPLE_RATE` |
| FR-10 | Prompt Lab: single-turn templates with editable sampling parameters. | `ui/llmsingleturn/*` |
| FR-11 | Agent Skills: load skills on demand; run JS skills, Android intents, and MCP tools; render image and webview results. | `AgentChatTask`, `tools/*` |
| FR-12 | Users can add skills from a URL or a local folder, enable or disable them, edit them, and test them. | `SkillManager`, `SkillManagerBottomSheet` |
| FR-13 | Users can add MCP servers (StreamableHTTP, optional header auth) and control per-tool permission. | `McpManagerViewModel`, `RunMcpTool` |
| FR-14 | Ask the user for consent before an MCP tool call unless that tool is set to always-allow. | `AskMcpToolCallPermissionAction` |
| FR-15 | Natural-language and image-to-image search over the user's photos or a sample album. | `customtasks/smartalbum/*` |
| FR-16 | Find moments in videos by description; trim, save, and export clips. | `customtasks/videomomentfinder/*` |
| FR-17 | Segment objects in photos and build collages. | `customtasks/scrapbook/*` |
| FR-18 | Offline device actions and a mini-game driven by function calling. | `mobileactions/*`, `tinygarden/*` |
| FR-19 | Persist chat history per task and let users resume or delete sessions. | `LlmSessionManager`, `ChatSessionRepository` |
| FR-20 | Benchmark LiteRT-LM models and keep the results. | `ui/benchmark/*` |
| FR-21 | Deep links open a task or model; an external query must be confirmed by the user before it is sent. | `GalleryNavGraph`, `ExternalPromptConfirmationDialog` |
| FR-22 | The user can opt out of analytics. | `Settings.disable_firebase_analytics` |

## 4. Non-functional requirements

| Id | Category | Requirement | Status / evidence |
|----|----------|-------------|-------------------|
| NFR-01 | Privacy | Prompts, images, audio, and model outputs are processed on-device only. | **[current]** No inference network calls. Remote MCP and skills are user-opt-in. |
| NFR-02 | Offline | Every task works offline once its model is downloaded. MCP and remote skills are the exceptions. | **[current]** |
| NFR-03 | Platform | Android 12+ (API 31). Targets API 37. | `minSdk 31`, `targetSdk 37` |
| NFR-04 | Device fit | Warn when RAM is below a model's `minDeviceMemoryInGb` (6–12 GB for current models). | `MemoryWarning` |
| NFR-05 | Performance | Choose the best available accelerator (GPU/NPU/TPU) with a CPU fallback. Record TTFT, prefill/decode speed, and memory. | `BackendSpec`, `common/metrics/*` |
| NFR-06 | Reliability | Downloads survive app kill (WorkManager foreground) and resume over HTTP Range. | `DownloadWorker` |
| NFR-07 | Reliability | The catalog loads from cache if GitHub is unreachable. | `readModelAllowlistFromDisk` |
| NFR-08 | Security | Untrusted inputs (deep-link queries, MCP calls, skill installs) need explicit user consent. | Disclaimers and permission dialogs |
| NFR-09 | Security | Credentials (HF tokens, skill secrets, MCP headers) are stored only on the device. | **[current]** Stored, but **unencrypted**. See [10-security-privacy](10-security-privacy.md). |
| NFR-10 | Accessibility | Content descriptions on icons, semantic headings, and support for dynamic text. | Partial **[inferred]**: `cd_*` strings and `semantics { heading() }` exist. No audit found. |
| NFR-11 | Localization | All user-facing strings in resources with translator notes. | Mostly. About 17 files have hard-coded text. |
| NFR-12 | Maintainability | Features plug in without editing central code (Hilt `@IntoSet`). | **[current]** |
| NFR-13 | Quality | Automated tests gate merges. | **Not met.** No tests exist (see [06-testing](06-testing.md)). **[proposed]** |
| NFR-14 | Observability | Opt-in, anonymous usage and performance telemetry. | Firebase Analytics (absent in OSS builds) |
