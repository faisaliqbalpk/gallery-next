---
id: build-ci-release
title: Build, CI, and Release
doc_type: release
status: mixed
app_version: 1.0.20
last_reviewed: 2026-10-24
audience: [engineers, release-managers, ai-agents]
related: [tech-stack, testing, how-to-guides, security-privacy]
sources:
  - Android/src/app/build.gradle.kts
  - .github/workflows/build_android.yaml
  - .github/workflows/static.yml
  - model_allowlists/
  - README.md
---

# Build, CI, and Release

## 1. Build configuration **[current]**

| Setting | Value |
|---------|-------|
| `applicationId` | `com.google.aiedge.gallery` (differs from the `namespace` `com.google.ai.edge.gallery`) |
| `versionCode` / `versionName` | `46` / `1.0.20` |
| Build types | `debug` (default), `release` (`isMinifyEnabled = false`, **signed with the debug key**, references a missing `proguard-rules.pro`) |
| Build features | Compose, BuildConfig (`FEEDBACK_API_KEY = ""`) |
| Manifest placeholders | `appAuthRedirectScheme` (placeholder), `applicationName`, `appIcon` |
| Packaging | `pickFirsts **/liblitertlm_jni.so` |
| Protobuf | protoc 4.26.1, `java` + `kotlin` lite builtins |

Common tasks (from `Android/src/`):

| Command | Result |
|---------|--------|
| `./gradlew assembleDebug` / `installDebug` | Debug APK / install on a connected device |
| `./gradlew assembleRelease` | Release APK (debug-signed); what CI builds |
| `./gradlew testDebugUnitTest` | Runs nothing today (no test sources) |
| `./gradlew lintDebug` | Android Lint (not run in CI) |

## 2. CI workflows **[current]**

| Workflow | Trigger | Steps | Gaps |
|----------|---------|-------|------|
| `build_android.yaml` — "Build Android APK" | push/PR to `main` touching `Android/**`; manual | checkout (v3) → Temurin JDK 21 → install SDK platform 37 → `./gradlew assembleRelease` | No tests, lint, or artifact upload. Doesn't run for allowlist or skill changes. |
| `static.yml` — "Deploy static content to Pages" | push to `main` touching `skills/**`; manual | Stage `skills/` → `staging/skills/` → deploy to GitHub Pages | No validation of `SKILL.md` |

There is **no CI for `model_allowlists/`**, yet those files change production behaviour as soon as they're merged.

## 3. Versioning **[current]**

- `versionName` uses `MAJOR.MINOR.PATCH`, and git tags match it (`1.0.16` … `1.0.20`). `versionCode` goes up by one integer per release.
- The model catalog is versioned **per app release**: the app requests `model_allowlists/<versionName with "." → "_">.json`.
  Allowlists exist for `1_0_4` … `1_0_20`, plus `ios_1_0_0.json` for the iOS app.
- Old app versions keep reading their own allowlist file, so a model can be retired for one version without affecting another.

## 4. Release process

Steps 1–6 are reconstructed from repository evidence **[inferred]**. Steps 7–8 are **[proposed]**.

1. Bump `versionCode` and `versionName` in `Android/src/app/build.gradle.kts`.
2. Create `model_allowlists/<new_version>.json`. Usually you copy the previous file, then add or adjust models.
   **Without this file, a fresh install of the new version has no model list.** (It falls back to the disk cache, and a
   new install has none.)
3. Make sure built-in skills in `assets/skills/` are up to date.
4. Merge to `main`. CI builds the release APK.
5. Tag the release with `versionName` and publish the APK on GitHub Releases (the README links to "latest release").
6. Publish the store builds through Google Play, the App Store (iOS), and the macOS DMG. Signing for these happens
   outside this repo.
7. **[proposed]** Run the release checklist in [06-testing §B.4](06-testing.md#b4-release-test-checklist-manual-per-release).
8. **[proposed]** Add CI gates: unit tests, lint, allowlist JSON validation, `SKILL.md` validation, and a check that
   `model_allowlists/<versionName>.json` exists for the version in `build.gradle.kts`.

## 5. Distribution channels

| Channel | Source |
|---------|--------|
| Google Play (`com.google.ai.edge.gallery` listing) | Root README link |
| GitHub Releases APK | Root README link |
| App Store (iOS 17+) | Root README link (source not in this repo) |
| macOS DMG | Root README link (source not in this repo) |

## 6. Environments and configuration

| Concern | OSS build default | To customise |
|---------|-------------------|--------------|
| Hugging Face OAuth | Placeholders, so gated downloads fail | [08-how-to-guides §2](08-how-to-guides.md#2-configure-hugging-face-oauth) |
| Firebase | Off (`apply false`, no `google-services.json`) | [08-how-to-guides §11](08-how-to-guides.md#11-enable-firebase-analytics-and-fcm) |
| Featured skills | Off (`SKILL_ALLOWLIST_URL = ""`) | Set the URL in `skills/SkillManager.kt` |
| Feedback API | `FEEDBACK_API_KEY = ""` | `buildConfigField` in `build.gradle.kts` |
| Model catalog | GitHub `main` | Local test file (`/data/local/tmp/model_allowlist_test.json`) |
