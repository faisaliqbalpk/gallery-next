---
id: agents-md
title: Instructions for AI Coding Agents
doc_type: agent-instructions
status: current
app_version: 1.0.20
last_reviewed: 2026-10-24
audience: [ai-agents]
related: [docs-index, architecture, code-conventions, testing, security-privacy, known-issues]
---

# AGENTS.md

Instructions for AI coding assistants (Claude Code, Codex, Copilot, Gemini, etc.) working in this repository.
Human-oriented documentation lives in [`Docs/`](Docs/README.md). Start there for anything beyond a small change.

## What this repo is

Google AI Edge Gallery: an Android app (Kotlin, Jetpack Compose, Hilt) that runs open models **on-device** with
LiteRT-LM, Android AICore, and MediaPipe. It has a single Gradle module, `:app`, under `Android/src/`.
This is a public mirror of an internal codebase, and external code contributions are not accepted ([CONTRIBUTING.md](CONTRIBUTING.md)).

## Orient yourself

| Need | Read |
|------|------|
| Layers, packages, startup, navigation | [Docs/01-architecture.md](Docs/01-architecture.md) |
| In-app agent, tools, skills, MCP, intents | [Docs/02-agents.md](Docs/02-agents.md) |
| Model catalog, downloads, runtimes, persistence | [Docs/04-technical-design.md](Docs/04-technical-design.md) |
| Style and patterns | [Docs/07-code-conventions.md](Docs/07-code-conventions.md) |
| Recipes (add a task, model, skill, intent) | [Docs/08-how-to-guides.md](Docs/08-how-to-guides.md) |
| Open defects and risks | [Docs/14-known-issues.md](Docs/14-known-issues.md) |

Kotlin sources: `Android/src/app/src/main/java/com/google/ai/edge/gallery/` (written `gallery/` in the docs).
If `graphify-out/graph.json` exists, a knowledge graph of the repo is available (`graphify query "<question>"`).
It is gitignored and optional.

## Build and verify

```bash
cd Android/src
./gradlew assembleDebug        # compile check; the minimum verification for any Kotlin change
./gradlew installDebug         # run on a connected Android 12+ device
```

- **There are no automated tests** (no `src/test` or `src/androidTest`). Don't claim tests passed. Report that the
  build compiled, and say what you verified manually, if anything.
- CI (`.github/workflows/build_android.yaml`) only runs `./gradlew assembleRelease`.
- If you add tests, put JVM tests in `Android/src/app/src/test/java/...`. Priority targets are in [Docs/06-testing.md](Docs/06-testing.md#b2-first-tests-to-write-highest-value).

## Rules

1. **License header.** Every new `.kt`, `.kts`, `.xml`, or `.proto` file starts with the Apache-2.0 header (`Copyright <year> Google LLC`), copied from a neighbouring file.
2. **Style.** 2-space indent, ~100 columns, trailing commas, ktfmt Google style. `private const val TAG = "AG<ClassName>"`. Interface + `Default<Impl>`.
3. **New feature = new task.** Implement `CustomTask` and register it with a Hilt `@Provides @IntoSet` module (see `customtasks/examplecustomtask/`). Don't hard-code features into navigation or home.
4. **State.** ViewModels use `@HiltViewModel`, a private `MutableStateFlow`, a public `asStateFlow()`, and `update { it.copy(...) }`. User-visible strings go in `res/values/strings.xml` with a `description` and `[CHAR_LIMIT]`.
5. **Async.** Inject dispatchers (`@IoDispatcher`, etc.). Don't add `runBlocking` on the main thread. The model lifecycle callback convention is `onDone("")` for success and `onDone("<message>")` for an error.
6. **Persistence.** Proto DataStore schema changes must be additive. Use new tag numbers and never reuse old ones.
7. **No secrets.** Leave `ProjectConfig.clientId`/`redirectUri` and `appAuthRedirectScheme` as `REPLACE_WITH_…` placeholders. Never add `google-services.json`, API keys, or tokens. Don't log secrets or tokens (existing violations: KI-04, KI-05).
8. **The model catalog is production.** `model_allowlists/<version>.json` is fetched live by installed apps from `main`. Edit it only when asked. Keep the JSON valid, and don't change files for older versions unless explicitly told to.
9. **Skills.** Built-in skills live in `Android/src/app/src/main/assets/skills/`. Keep `skills/built-in/` identical for any skill that exists in both places.
10. **Security-sensitive files.** Changes to `ui/common/GalleryWebView.kt`, `tools/*`, `intents/IntentHandler.kt`, `data/DataStoreRepository.kt`, `worker/DownloadWorker.kt`, or `customtasks/agentchat/McpManager*` need extra care. Read [Docs/10-security-privacy.md](Docs/10-security-privacy.md) first. Keep the human-consent steps (MCP permission, external-prompt confirmation, composer-only send intents).
11. **Docs stay in sync.** Each doc in `Docs/` lists its `sources` in front matter. If you change one of those files in a way that affects the doc, update the doc and its `last_reviewed` date in the same change.

## Gotchas

- Context compaction code exists but is off (`Config.isContextCompactEnabled() = false`).
- The featured-skills list is empty (`SKILL_ALLOWLIST_URL = ""`).
- `AgentTools` is a singleton whose `lateinit` fields are set by `AgentChatScreen` (KI-07). Don't rely on them outside that screen's lifetime.
- Only one LLM session is active app-wide (`LlmSessionManager`).
- `Model.runtimeHelper` picks AICore vs LiteRT-LM. Tests can override every model through `testingModelHelper`.
- Local catalog override: `adb push <file>.json /data/local/tmp/model_allowlist_test.json`.
