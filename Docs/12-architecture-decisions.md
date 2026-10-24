---
id: architecture-decisions
title: Architecture Decision Records (reconstructed)
doc_type: adr
status: current
app_version: 1.0.20
last_reviewed: 2026-10-24
audience: [engineers, architects, ai-agents]
related: [architecture, technical-design, agents]
sources:
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/customtasks/common/CustomTask.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/ui/modelmanager/ModelManagerViewModel.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/runtime/ModelHelperExt.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/di/AppModule.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/agent/sessions/LlmSessionManager.kt
  - skills/README.md
  - mcp/README.md
---

# Architecture Decision Records

> **These ADRs are reconstructed.** No decision records exist in the repository. Each entry describes a decision
> that is visible in the code, with the likely context. The **Evidence** line is what the code shows. The
> **Context** is inferred. Treat them as a baseline, and record new decisions with the template at the end.

| ADR | Title | Status |
|-----|-------|--------|
| [ADR-001](#adr-001-on-device-only-inference) | On-device-only inference | Accepted |
| [ADR-002](#adr-002-features-as-hilt-multibound-tasks) | Features as Hilt-multibound tasks | Accepted |
| [ADR-003](#adr-003-static-versioned-model-catalog-on-github) | Static, versioned model catalog on GitHub | Accepted |
| [ADR-004](#adr-004-pluggable-runtimes-keyed-by-runtimetype) | Pluggable runtimes keyed by `RuntimeType` | Accepted |
| [ADR-005](#adr-005-proto-datastore-for-app-state) | Proto DataStore for app state | Accepted |
| [ADR-006](#adr-006-workmanager-foreground-workers-for-long-jobs) | WorkManager foreground workers for long jobs | Accepted |
| [ADR-007](#adr-007-skills-as-skillmd--javascript-in-a-hidden-webview) | Skills as `SKILL.md` + JavaScript in a hidden WebView | Accepted |
| [ADR-008](#adr-008-mcp-over-streamablehttp-only) | MCP over StreamableHTTP only | Accepted |
| [ADR-009](#adr-009-single-active-model-independent-llm-session) | Single active, model-independent LLM session | Accepted |
| [ADR-010](#adr-010-human-in-the-loop-for-side-effects) | Human-in-the-loop for side effects | Accepted |

---

## ADR-001: On-device-only inference

- **Context:** The product's purpose is to show private, offline generative AI on phones.
- **Decision:** All inference runs locally (LiteRT-LM, AICore, MediaPipe). The network is used only for catalog,
  model files, opt-in tools, and telemetry.
- **Consequences:** Model size and RAM limit features (`minDeviceMemoryInGb`), and hardware acceleration becomes
  critical (GPU/NPU/TPU paths). No backend to run, but no server-side safety layer either.
- **Evidence:** No inference HTTP clients. README ("100% On-Device Privacy").

## ADR-002: Features as Hilt-multibound tasks

- **Context:** Many independent demos, each with its own model lifecycle and UI, have to coexist and change on their own.
- **Decision:** The `CustomTask` interface. Each feature contributes `@IntoSet` to `Set<CustomTask>`. Navigation and
  model management are generic.
- **Consequences:** Adding a feature needs no central edits (except home ordering). Tasks share one global
  `ModelManagerViewModel`. Built-in and custom tasks use the same mechanism.
- **Evidence:** `customtasks/common/CustomTask.kt` and 11 `*Module` files with `@IntoSet`.

## ADR-003: Static, versioned model catalog on GitHub

- **Context:** Models change faster than app releases, and there is no backend.
- **Decision:** The app fetches `model_allowlists/<versionName>.json` from `raw.githubusercontent.com` (`main` branch),
  caches it, and falls back to the cache.
- **Consequences:** No server is needed, and models can be updated without a release. But a merge to `main` is a
  production deploy, the file must exist before a new version ships, and the file is neither signed nor validated in CI
  (see [10-security-privacy](10-security-privacy.md) SEC-03).
- **Evidence:** `ALLOWLIST_BASE_URL`, `getAllowlistUrl()`, `loadModelAllowlist()`.

## ADR-004: Pluggable runtimes keyed by `RuntimeType`

- **Context:** Some devices ship system models (AICore). Most models are app-downloaded LiteRT-LM bundles.
- **Decision:** `RuntimeType` (`litert_lm`, `aicore`, `unknown`) carries capability flags. Runtime selection goes
  through `Model.runtimeHelper`, and download through a Hilt map of `ModelDownloader` keyed by `@RuntimeTypeKey`.
- **Consequences:** A new runtime means a new `LlmModelHelper`, a new `ModelDownloader`, one map entry, and a
  `runtimeHelper` branch. Behaviour (auto-download, deletable, top-level download) is data-driven.
- **Evidence:** `data/ModelEnums.kt`, `di/AppModule.kt`, `runtime/ModelHelperExt.kt`.

## ADR-005: Proto DataStore for app state

- **Context:** Typed, transactional local state across many features.
- **Decision:** One Proto DataStore per concern (`settings`, `user_data`, `skills`, `mcp_servers`, `cutouts`,
  `benchmark_results`, `video_moment_finder`, `onboarding`), each exposed as a Hilt singleton.
- **Consequences:** Schema changes are additive through proto tags. No SQL except the MediaPipe vector stores.
  Secrets landed in the same plaintext store (see SEC-02).
- **Evidence:** `app/src/main/proto/*.proto`, `di/AppModule.kt`.

## ADR-006: WorkManager foreground workers for long jobs

- **Context:** Multi-GB downloads and photo indexing must survive navigation and process death.
- **Decision:** `DownloadWorker` and `SmartAlbumIndexingWorker` run as foreground `dataSync` workers with unique work
  names. Downloads resume with HTTP `Range` from `.gallerytmp` files.
- **Consequences:** Reliable progress and resume. Progress goes through WorkManager `Data`, and so does the access token.
- **Evidence:** `worker/DownloadWorker.kt`, `data/DownloadRepository.kt`.

## ADR-007: Skills as `SKILL.md` + JavaScript in a hidden WebView

- **Context:** On-device models can't spawn processes or run Python. Skills need to be shareable and portable across Android and iOS.
- **Decision:** A skill is Markdown instructions plus an optional `scripts/index.html` that exposes
  `window.ai_edge_gallery_get_result`. The model loads instructions with `load_skill` and runs code with `run_js`.
  Native capabilities are reached through `run_intent`.
- **Consequences:** Skills are portable and easy to author and host (GitHub Pages). JavaScript runs inside the app's
  WebView, so origin isolation matters (SEC-01). Quality depends on how well a small model follows the instructions.
- **Evidence:** `skills/README.md`, `tools/RunJsTool.kt`, `skills/SkillManager.kt`.

## ADR-008: MCP over StreamableHTTP only

- **Context:** Mobile apps can't host stdio MCP servers.
- **Decision:** Support remote MCP servers over StreamableHTTP (Ktor + SSE), authenticated with custom headers. OAuth is deferred.
- **Consequences:** Local servers need a bridge (supergateway) and a tunnel. A per-tool consent model was needed.
- **Evidence:** `McpManagerViewModel` (`StreamableHttpClientTransport`), `mcp/README.md`.

## ADR-009: Single active, model-independent LLM session

- **Context:** Phone memory allows one large model at a time. Users switch models mid-conversation.
- **Decision:** `LlmSessionManager` assumes one active session. A session id identifies history, not a model.
- **Consequences:** Simpler resource management. Running two models concurrently isn't supported.
- **Evidence:** KDoc of `agent/sessions/LlmSessionManager.kt`.

## ADR-010: Human-in-the-loop for side effects

- **Context:** Small models make mistakes, and tools act on the user's data and device.
- **Decision:** "Send" intents open composers instead of sending. MCP calls need consent (Deny / Allow once / Always).
  External prompts from deep links must be confirmed. Adding a skill or server shows a disclaimer.
- **Consequences:** Safer, with more taps. `always_allow` is the user's explicit opt-out.
- **Evidence:** `IntentHandler`, `RunMcpTool`, `ExternalPromptConfirmationDialog`, `Add*DisclaimerDialog`.

---

## Template for new ADRs

```markdown
## ADR-0NN: <short decision title>

- **Status:** Proposed | Accepted | Superseded by ADR-0MM
- **Date:** YYYY-MM-DD
- **Context:** <forces, constraints, problem>
- **Decision:** <what we will do>
- **Alternatives considered:** <options and why rejected>
- **Consequences:** <positive, negative, follow-ups>
- **Evidence / links:** <code paths, PRs, issues>
```
