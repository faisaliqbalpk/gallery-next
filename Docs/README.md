---
id: docs-index
title: Google AI Edge Gallery — Engineering Documentation
doc_type: index
status: current
app_version: 1.0.20
last_reviewed: 2026-10-24
audience: [engineers, ai-agents]
related: [architecture, agents, functional-design, technical-design, tech-stack, testing, code-conventions, how-to-guides, requirements, security-privacy, build-ci-release, architecture-decisions, glossary, known-issues]
sources:
  - README.md
  - DEVELOPMENT.md
---

# Google AI Edge Gallery — Engineering Documentation

Google AI Edge Gallery is an Android app (label "Edge Gallery") that runs open-weight generative AI models
**fully on-device** with LiteRT-LM, Android AICore, and MediaPipe. This folder describes how it is built.
For what the product does from a user's point of view, see the root [README](../README.md).

## Documents

| # | Document | `id` | What it answers |
|---|----------|------|-----------------|
| 01 | [Architecture](01-architecture.md) | `architecture` | How the app is layered, how modules connect, startup and navigation |
| 02 | [Agents](02-agents.md) | `agents` | How the in-app agent runtime, tools, Agent Skills, MCP, and intents work |
| 03 | [Functional design](03-functional-design.md) | `functional-design` | What each feature does, its user flow, inputs, limits, and models |
| 04 | [Technical design](04-technical-design.md) | `technical-design` | Model catalog, download, runtimes, sessions, persistence, workers, telemetry |
| 05 | [Tech stack](05-tech-stack.md) | `tech-stack` | Languages, build tooling, libraries, and versions |
| 06 | [Testing](06-testing.md) | `testing` | Current test state (none automated) and the proposed strategy |
| 07 | [Code conventions](07-code-conventions.md) | `code-conventions` | Conventions observed in the code |
| 08 | [How-to guides](08-how-to-guides.md) | `how-to-guides` | Step-by-step recipes for common engineering tasks |
| 09 | [Requirements](09-requirements.md) | `requirements` | Goals, non-goals, functional and non-functional requirements |
| 10 | [Security & privacy](10-security-privacy.md) | `security-privacy` | Permissions, data at rest, network surface, risks |
| 11 | [Build, CI & release](11-build-ci-release.md) | `build-ci-release` | Build variants, CI workflows, versioning, release steps |
| 12 | [Architecture decisions](12-architecture-decisions.md) | `architecture-decisions` | ADRs reconstructed from the code |
| 13 | [Glossary](13-glossary.md) | `glossary` | Domain and project terms |
| 14 | [Known issues](14-known-issues.md) | `known-issues` | Defects, risks, and doc/code mismatches found during review |
| — | [AGENTS.md](../AGENTS.md) | `agents-md` | Instructions for AI coding assistants working in this repo |

Reading order for a new engineer: 01 → 05 → 03 → 04 → 02 → 07 → 08. For a reviewer or auditor: 09 → 10 → 14.

## Existing docs (not moved)

| File | Topic |
|------|-------|
| [README.md](../README.md) | Product overview and install links |
| [DEVELOPMENT.md](../DEVELOPMENT.md) | Hugging Face OAuth setup for local builds |
| [Function_Calling_Guide.md](../Function_Calling_Guide.md) | Adding a Mobile Actions function |
| [Bug_Reporting_Guide.md](../Bug_Reporting_Guide.md) | Capturing Android bug reports |
| [skills/README.md](../skills/README.md) | Authoring Agent Skills (`SKILL.md`, JS skills) |
| [mcp/README.md](../mcp/README.md) | Connecting MCP servers |

## Conventions used in these docs

### Front matter schema

Every document starts with YAML front matter so tools can index it:

| Key | Type | Meaning |
|-----|------|---------|
| `id` | string | Stable identifier. Other docs refer to it in `related`. |
| `title` | string | Human title. |
| `doc_type` | enum | `index`, `architecture`, `agents`, `functional-design`, `technical-design`, `tech-stack`, `testing`, `code-conventions`, `how-to`, `requirements`, `security`, `release`, `adr`, `glossary`, `known-issues`, `agent-instructions` |
| `status` | enum | `current` = describes code as it is. `proposed` = recommendation, not implemented. `mixed` = both; each section is labelled. |
| `app_version` | string | `versionName` the doc was checked against (from `Android/src/app/build.gradle.kts`). |
| `last_reviewed` | date | ISO date of the last check against the code. |
| `audience` | list | Intended readers. |
| `related` | list | `id`s of related docs. |
| `sources` | list | Repo-relative paths the doc was derived from. Re-check these when updating the doc. |

### Path shorthand

Kotlin sources live under `Android/src/app/src/main/java/com/google/ai/edge/gallery/`.
In tables and prose this prefix is written as **`gallery/`**, e.g. `gallery/agent/AgentRuntimeExecutor.kt`.
Front matter `sources` always use full repo-relative paths.

### Labels in text

- **[current]** — verified in code at `app_version`.
- **[proposed]** — a recommendation; nothing in the repo implements it yet.
- **[inferred]** — deduced from indirect evidence (comments, naming, build files); not directly confirmed.
- **[to verify]** — could not be confirmed from the source in this repo.

### Keeping docs current

When a change touches a file listed in a doc's `sources`, update that doc and its `last_reviewed` date
in the same change. If you bump `versionName`, re-check `app_version` across docs.
