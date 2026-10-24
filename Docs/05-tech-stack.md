---
id: tech-stack
title: Technical Stack Overview
doc_type: tech-stack
status: current
app_version: 1.0.20
last_reviewed: 2026-10-24
audience: [engineers, ai-agents]
related: [architecture, build-ci-release, technical-design]
sources:
  - Android/src/gradle/libs.versions.toml
  - Android/src/app/build.gradle.kts
  - Android/src/build.gradle.kts
  - Android/src/settings.gradle.kts
  - Android/src/gradle.properties
  - Android/src/gradle/wrapper/gradle-wrapper.properties
  - .github/workflows/build_android.yaml
---

# Technical Stack Overview

Versions come from `Android/src/gradle/libs.versions.toml` unless noted. When you upgrade a library, update this table.

## 1. Platform and toolchain

| Item | Value | Source |
|------|-------|--------|
| Language | Kotlin 2.2.21 (`kotlin.code.style=official`) | toml |
| JVM target | Java 11 (`sourceCompatibility`/`targetCompatibility`, `jvmTarget`) | `app/build.gradle.kts` |
| JDK used by CI | Temurin 21 | `build_android.yaml` |
| Gradle | 9.2.1 (wrapper) | `gradle-wrapper.properties` |
| Android Gradle Plugin | 9.0.1 | toml |
| KSP | 2.3.6 | toml |
| compileSdk / targetSdk / minSdk | 37 / 37 / 31 (Android 12) | `app/build.gradle.kts` |
| Modules | `:app` only | `settings.gradle.kts` |
| Repositories | `google()`, `mavenCentral()` (`FAIL_ON_PROJECT_REPOS`) | `settings.gradle.kts` |
| Gradle JVM heap | `-Xmx2048m` | `gradle.properties` |

## 2. Gradle plugins

| Plugin | Purpose |
|--------|---------|
| `com.android.application` | App build |
| `org.jetbrains.kotlin.plugin.compose` | Compose compiler |
| `org.jetbrains.kotlin.plugin.serialization` (2.0.21) | kotlinx.serialization |
| `com.google.protobuf` (0.10.0, protoc 4.26.1, `java`+`kotlin` lite) | Proto codegen |
| `com.google.dagger.hilt.android` (2.60.1) | DI |
| `com.google.devtools.ksp` | Hilt and Moshi codegen |
| `com.google.android.gms.oss-licenses-plugin` (0.11.0) | Licenses screen |
| `com.google.gms.google-services` (4.4.3) | **`apply false`**. Needs `google-services.json`, which is not in the repo. |

## 3. Libraries by concern

| Concern | Library | Version |
|---------|---------|---------|
| UI | Jetpack Compose BOM | 2026.02.00 |
| | Material 3, Material Icons Extended | BOM / 1.7.8 |
| | Navigation Compose | 2.8.9 |
| | Activity Compose | 1.10.1 |
| | Core SplashScreen | 1.2.0-beta01 |
| | compose-richtext (Markdown, Material3 + CommonMark) | 1.0.0-alpha02 |
| | Coil Compose | 2.7.0 |
| Lifecycle | lifecycle-runtime-ktx, lifecycle-process | 2.8.7 |
| DI | Hilt Android, Hilt Navigation Compose | 2.60.1 / 1.3.0 |
| LLM runtime | LiteRT-LM Android (`com.google.ai.edge.litertlm:litertlm-android`) | 0.18.0 |
| | ML Kit GenAI Prompt (AICore) | 1.0.0-beta2 |
| Classical ML | Play Services TFLite Java / GPU / Support | 16.4.0 |
| | MediaPipe Tasks Vision (InteractiveSegmenter) | 1.0.0 |
| | MediaPipe Tasks Retrieval (UniversalEmbedder, SemanticRetriever, SqliteVectorStore) | 1.1.0 |
| Agent / MCP | MCP Kotlin SDK (`io.modelcontextprotocol:kotlin-sdk`) | 0.8.0 |
| | Ktor Client Core + Android (SSE) | 3.4.3 |
| Persistence | DataStore (proto) | 1.1.7 |
| | Protobuf Java/Kotlin Lite | 4.26.1 |
| Background | WorkManager (`work-runtime-ktx`) | 2.12.0 |
| Serialization | Gson | 2.14.0 |
| | Moshi Kotlin + codegen | 1.15.2 |
| | kotlinx-serialization-json | 1.7.3 |
| | kotlin-reflect, kotlin-metadata-jvm (forced) | 2.4.0 |
| Media | CameraX core/camera2/lifecycle/view | 1.4.2 |
| | Media3 ExoPlayer / Transformer / UI | 1.11.1 |
| | ExifInterface | 1.4.1 |
| Web | AndroidX WebKit (`WebViewAssetLoader`) | 1.14.0 |
| Auth | AppAuth (`net.openid:appauth`) | 0.11.1 |
| Security | AndroidX Security Crypto | 1.1.0 (**declared, unused**) |
| Firebase | BoM, Analytics, Messaging | 33.16.0 |
| Licenses | play-services-oss-licenses | 17.5.1 |

Test dependencies are declared but there are **no test sources** (see [06-testing](06-testing.md)):
JUnit 4.13.2, AndroidX JUnit 1.2.1, Espresso 3.6.1, Compose `ui-test-junit4`, `hilt-android-testing`.

## 4. Data and assets formats

| Format | Where |
|--------|-------|
| `.litertlm` | LiteRT-LM model bundles: all LLMs and EmbeddingGemma-2 in the 1.0.20 allowlist |
| `.task` | MediaPipe task bundles: the Scrapbook segmenter (`interactive_segmentation.task`), legacy LLMs, and importable models |
| `.zip` | Optional extra data (e.g. EmbeddingGemma-2 `samples.zip` sample album), unzipped after download |
| Protobuf (`*.proto`) | DataStore schemas, Hugging Face model cache, metrics |
| JSON | Model allowlists, skill allowlist, JS skill results, intent parameters |
| `SKILL.md` + HTML/JS | Agent Skills |

## 5. External services

| Service | Use | Required? |
|---------|-----|-----------|
| GitHub raw (`raw.githubusercontent.com`) | Model allowlist | Yes for the first catalog load; cached afterwards |
| GitHub API (`api.github.com`) | Latest-release check | No |
| GitHub Pages | Hosting `skills/` (from the `static.yml` workflow) | Only for remote/featured skills |
| Hugging Face (`huggingface.co`) | Model files, model search API, OAuth | Yes for downloads |
| Google Play services / AICore | AICore models, TFLite | Only on supported devices |
| Firebase Analytics / FCM | Telemetry, push | No (absent in OSS builds) |
| User-configured MCP servers and skill URLs | Agent tools | No |

## 6. Native and hardware acceleration

- LiteRT-LM backends: CPU, GPU (OpenCL), NPU/TPU (vendor libraries; per-SoC model files via `socToModelFiles`).
- The manifest declares `android.hardware.npu` as optional, plus optional native libraries `libOpenCL.so`, `libcdsprpc.so`
  (Qualcomm DSP), `libedgetpu_*.so` (Google Tensor), `libvndksupport.so`.
- The MediaPipe embedder supports NPU, GPU, and CPU delegates.
