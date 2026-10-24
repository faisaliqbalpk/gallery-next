---
id: known-issues
title: Known Issues, Risks, and Doc/Code Mismatches
doc_type: known-issues
status: current
app_version: 1.0.20
last_reviewed: 2026-10-24
audience: [engineers, reviewers, ai-agents]
related: [security-privacy, testing, agents, technical-design]
sources:
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/ui/modelmanager/ModelManagerViewModel.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/ui/common/GalleryWebView.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/tools/RunJsTool.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/worker/DownloadWorker.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/MainActivity.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/customtasks/agentchat/AgentChatScreen.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/customtasks/agentchat/AgentChatTaskModule.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/customtasks/agentchat/AgentTools.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/data/Categories.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/data/Config.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/data/ModelAllowlist.kt
  - Android/src/app/src/main/java/com/google/ai/edge/gallery/customtasks/examplecustomtask/ExampleCustomTask.kt
  - skills/
---

# Known Issues, Risks, and Doc/Code Mismatches

These were found while reviewing the code for this documentation set. **No fixes have been applied.** Each item has a
stable id. Use it in issues and commits, and update the **State** column when an item is resolved.

**Type:** `bug` · `security` · `risk` (latent defect) · `mismatch` (docs vs code) · `debt` · `config`.
**Confidence:** `confirmed` (read in code) · `to verify` (needs a runtime check).

## 1. Code

| Id | Type | Sev. | Summary | Evidence | Confidence | State |
|----|------|------|---------|----------|------------|-------|
| <a id="ki-01"></a>KI-01 | bug | Med | HF token expiry "5-minute buffer" subtracts `5 * 60` **ms** (300 ms). Fix: `5 * 60 * 1000L`. | `ModelManagerViewModel.kt:1104` | confirmed | open |
| <a id="ki-02"></a>KI-02 | security | High | The WebView serves all of `filesDir` (including `datastore/user_data.pb` with tokens, secrets, and chat history) on the origin that local and imported skills run on. | `GalleryWebView.kt:69-85` | to verify | open |
| <a id="ki-03"></a>KI-03 | security | Med | Tokens, skill secrets, and MCP headers are stored unencrypted. `security-crypto` is declared but unused. Backup rules are unmodified samples with `allowBackup=true`. | `DataStoreRepository.kt`, `res/xml/*rules.xml` | confirmed | open |
| <a id="ki-04"></a>KI-04 | bug / security | Low | `RunJsTool` logs `secret.substring(0, 3)`. It leaks a prefix and throws `StringIndexOutOfBoundsException` for secrets of 1–2 characters. | `RunJsTool.kt:104` | confirmed | open |
| <a id="ki-05"></a>KI-05 | security | Low | `DownloadWorker` logs the first 10 characters of the HF token. The token is also passed through WorkManager input `Data`, which is persisted. | `DownloadWorker.kt:112,144` | confirmed | open |
| <a id="ki-06"></a>KI-06 | security | Low | `MainActivity` logs every intent extra (a "Debug" dump), in release builds too. | `MainActivity.kt` | confirmed | open |
| <a id="ki-07"></a>KI-07 | risk | Med | `AgentTools` is a Hilt `@Singleton`, but its `lateinit` fields (`context`, `dataStoreRepository`, `mcpManagerViewModel`, `taskId`) are assigned from the `AgentChatScreen` composable. The singleton then holds a screen-scoped ViewModel, which can leak or go stale. `AgentChatTask.initializeModelFn` reads `agentTools.mcpManagerViewModel`, so it would throw `UninitializedPropertyAccessException` if it ran before the screen composed. `runMcpTool` is `by lazy` and captures the first `taskId`. | `AgentChatScreen.kt:137-141`, `AgentTools.kt`, `AgentChatTaskModule.kt` | to verify | open |
| <a id="ki-08"></a>KI-08 | risk | Low | Tool names are inconsistent. The system prompt says `load_skill` and `run_intent` (snake_case) but `runMcpTool` (camelCase). The Kotlin methods are `loadSkill`, `runJs`, `runIntent`, `runMcpTool`. Whichever convention LiteRT-LM exposes, one of the names in the prompt is wrong. | `AgentChatTaskModule.kt` prompts, `tools/*` | to verify | open |
| <a id="ki-09"></a>KI-09 | config | Info | Context compaction is fully built (compactor, UI banner, per-task setting) but hard-disabled: `Config.isContextCompactEnabled()` returns `false`. | `Config.kt:180` | confirmed | open |
| <a id="ki-10"></a>KI-10 | debt | Info | `ModelAllowlist.flags` (remote flag overrides) is parsed but never read. | `ModelAllowlist.kt` | confirmed | open |
| <a id="ki-11"></a>KI-11 | bug | Low | `Category.CLASSICAL_ML` uses `R.string.category_llm` as its label. `category_classical_ml` exists. | `Categories.kt:42` | confirmed | open |
| <a id="ki-12"></a>KI-12 | bug | Low | `ExampleCustomTask` `docUrl` and `sourceCodeUrl` are missing `/blob/main/`, so the links 404. | `ExampleCustomTask.kt:75,77` | confirmed | open |
| <a id="ki-13"></a>KI-13 | risk | Low | `DownloadWorker` catches only `IOException`. The zip-slip `SecurityException` and other runtime errors fail the work with no error message for the UI. | `DownloadWorker.kt:344,359` | confirmed | open |
| <a id="ki-14"></a>KI-14 | debt | Low | `DefaultAgentRuntimeExecutor` ignores the `Attachment.ImageUri`, `AudioUri`, and `File` attachment types. | `DefaultAgentRuntimeExecutor.kt` | confirmed | open |
| <a id="ki-15"></a>KI-15 | risk | Low | DataStore reads and writes use `runBlocking`. Calls from the main thread can jank or ANR. | `DataStoreRepository.kt` | confirmed | open |
| <a id="ki-17"></a>KI-17 | security | Med | The model allowlist is fetched from GitHub `main` without a signature or pinned hashes, and downloaded models aren't hash-verified. Write access to `main` controls model URLs for every install of that version. | `ModelManagerViewModel.loadModelAllowlist`, `DownloadWorker` | confirmed | open |
| <a id="ki-16"></a>KI-16 | debt | Low | The `SKILL.md` front-matter parser is line-based, not YAML. Quoted, multi-line, or nested values parse incorrectly, and a `---` inside a value breaks the split. | `SkillManager.convertSkillMdToProto` | confirmed | open |

## 2. Build, CI, and configuration

| Id | Type | Sev. | Summary | Evidence | State |
|----|------|------|---------|----------|-------|
| KI-20 | debt | Med | No automated tests or lint in CI. Allowlist and skill changes ship with no validation. | [06-testing](06-testing.md), `.github/workflows/` | open |
| KI-21 | config | Info | The release build is debug-signed, `isMinifyEnabled = false`, and `proguard-rules.pro` is referenced but missing. | `app/build.gradle.kts` | open |
| KI-22 | config | Info | `SKILL_ALLOWLIST_URL = ""`, so the "community-featured skills" list described in `skills/README.md` is always empty in this build. | `SkillManager.kt:54` | open |
| KI-23 | config | Info | `applicationId` `com.google.aiedge.gallery` differs from the Play listing id `com.google.ai.edge.gallery` in the README. OSS builds therefore install next to the store app (probably intentional **[inferred]**). | `app/build.gradle.kts`, `README.md` | open |
| KI-24 | debt | Info | A stray `gallery/worker/AndroidManifest.xml` (targetSdk 35) sits inside the Java source tree and isn't used by Gradle. | `.../gallery/worker/AndroidManifest.xml` | open |
| KI-25 | debt | Info | The root `model_allowlist.json` (4 legacy `.task` models) isn't referenced by current code **[inferred]**. | `ModelManagerViewModel.ALLOWLIST_BASE_URL` | open |

## 3. Documentation vs code mismatches

Initially reported by the graph-extraction pass, then confirmed by direct reading.

| Id | Where | Docs say | Code does | State |
|----|-------|----------|-----------|-------|
| KI-30 | `skills/README.md:381` | Links `customtasks/agentchat/IntentHandler.kt` | The file is `intents/IntentHandler.kt` | open |
| KI-31 | `skills/featured/mood-music` | `SKILL.md` documents `genre_blend` and `bpm` | `scripts/index.html` sends only `genre`, `duration`, `energy` | open |
| KI-32 | `skills/built-in/qr-code` | `SKILL.md` doesn't mention `shape` | The script accepts `shape` (`square`/`circle`) | open |
| KI-33 | `skills/built-in/text-spinner` | — | `<title>Create github issue</title>` is left over, and `label` goes into the webview URL without `encodeURIComponent` | open |
| KI-34 | `skills/built-in/mood-tracker` | The dashboard shows when `show_dashboard` is requested | `showDashboard = input.show_dashboard \|\| action === 'get_history'`, so it always shows on history | open |
| KI-35 | `skills/featured/virtual-piano` | `SKILL.md`: `index.js` "returns the webview URL pointing to the GitHub-hosted UI" | Returns the relative `ui.html?v=<ts>` (served from the skill's `assets/`) | open |
| KI-36 | `skills/built-in/interactive-map/SKILL.md` vs `assets/skills/interactive-map/SKILL.md` | The published copy should match the bundled one | The two files differ | open |
| KI-37 | `DEVELOPMENT.md` | Links `app/build.gradle.kts` at an old commit (`c1b50e1…#L41-L44`) | The placeholder is now at `Android/src/app/build.gradle.kts:44-45` on `main` | open |
