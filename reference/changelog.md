# Changelog (User-Facing Summary)

This page summarizes notable release changes for operators and testers.

For full technical detail, see the project changelog in the plugin repository.

## Unreleased

<!-- Add new entries for the next release here. The release workflow copies this section into the GitHub release body when the tag is cut. Keep the heading in place even when the section is empty so contributors know where to append. -->


## 0.6.0 (2026-04-21)

Highlights:

* **NVIDIA NIM backend** — 10th backend, targets `integrate.api.nvidia.com`; configurable URL, model, API key, headers, and timeout.
* **MCP proxy-history preprocessing** — MCP tools that surface Burp's proxy history now run through a preprocessor: optional binary-content filter, per-response body size cap, max items per request, newest-first ordering, and an opt-in "raw / unpreprocessed" switch. Configurable under **Settings → MCP**.
* **Privacy-by-default** — default privacy mode is now `BALANCED` (cookies stripped, tokens redacted, hostnames preserved). Previous explicit choices are preserved; new installs and anyone who never chose a mode get redaction automatically.
* **Context preview dialog** — every right-click action that auto-captures context (proxy item, scanner issue, site-map node, Repeater) opens a modal showing the prompt, the active privacy mode, and the exact redacted JSON before anything is sent. **Cancel** aborts with no session created.
* **Custom prompt library** — right-click any request or scanner issue and pick **Custom prompts** to run a saved free-form prompt against that context, or **Custom…** to type an ad-hoc one. Saved prompts live under **Settings → Prompt Templates → Custom prompt library** (Add / Edit / Duplicate / Delete / Move Up / Move Down), are tagged per applicability (HTTP request, scanner issue, or both), and carry an *Show in context menu* toggle. The library persists as JSON (schema v3). Ad-hoc custom prompts skip the excerpt preview and go straight to the exact-send preview. All custom launches (and the existing canned ones) stamp `promptSource`, `contextKind`, and optional `promptId`/`promptTitle` into the audit log and prompt bundles so runs are reproducible and filterable with `jq`.
* **Expanded redaction coverage** — the token redactor now also strips `X-Auth-Token`, `X-Access-Token`, `X-Session-Token`, `X-CSRF-Token`, `X-Api-Secret`, `X-Client-Secret` (and their non-prefixed variants); `Basic <base64>` is replaced with `Basic [REDACTED]`; URL query parameters named `access_token`, `api_key`, `apikey`, `auth`, `token`, `key`, `secret`, `password`, `pwd`, `session`, `sid`, `code` have their value redacted in-place.
* **Scanner prompt hardening** — passive scanner (single + batch) and adaptive payload generator prompts now instruct the model to treat captured HTTP traffic and observed tech-stack / error patterns as untrusted data, not as instructions, even if a response body tries to override the prompt.
* **Vulnerability class cleanup** — removed the duplicate `RACE_CONDITION` entry; race-condition issues now use the single canonical `RACE_CONDITION_TOCTOU` class. `VulnClassInventoryTest` locks the 62-class count plus coverage of severity + remediation mappings.
* **JAR renamed** — build output is now `Custom-AI-Agent-<version>.jar` (was `Burp-AI-Agent-<version>.jar`). External scripts or download URLs that hard-code the old name must be updated.
* **Settings cache fix** — saving from the Settings tab now immediately reflects in right-click menus (library additions, prompt edits, etc.); no more Burp restart required.
* **Runtime defaults restored** — token budget and CLI idle-timeout defaults are explicit again after an earlier regression; MCP preprocess change detection now invalidates the tool schema so clients re-read the active shape.
* **Operational docs** — `docs/mcp-hardening.md` gained a *Credential Storage* section covering how the MCP bearer token and TLS keystore password are persisted and rotated.
* **Community and CI hygiene** — added `CODE_OF_CONDUCT.md`, `.github/dependabot.yml`, PR template, YAML-form issue templates (`bug_report.yml`, `feature_request.yml`, `config.yml`); build pipeline gained ktlint, JaCoCo coverage, a multi-OS matrix (`ubuntu-latest`, `macos-latest`, `windows-latest`), CycloneDX SBOM generation, SHA-256 checksum on the release JAR, and release notes extracted from the CHANGELOG section of the tagged version.

## 0.5.0 (2026-04-02)

Highlights:

* **Batch Analysis** — passive scanner groups 3-5 requests per AI call, reducing API costs by 60-70% with cross-request IDOR/BAC detection.
* **Structured Output (JSON Mode)** — HTTP backends (OpenAI, LM Studio, Ollama) now request guaranteed JSON output, eliminating parsing errors from markdown wrapping.
* **Persistent Prompt Cache** — AI results cached to disk (`~/.burp-ai-agent/cache/`) survive Burp restarts. Configurable TTL (1-168 hrs) and max size (10-500 MB).
* **Cross-Scanner Knowledge Base** — shared `ScanKnowledgeBase` feeds tech stack, auth patterns, and vulnerability signals between passive scanner, active scanner, and chat.
* **AI Adaptive Payloads** — active scanner generates context-aware payloads using AI based on detected tech stack and error patterns, with destructive-command safety filter.
* **Cache Normalization** — response fingerprints strip dynamic values (UUIDs, timestamps, tokens); endpoint keys sort query params and exclude cache-busting params.
* **New UI Settings** — batch size, persistent cache (enable/TTL/max), and adaptive payloads controls added to scanner settings panels.
* Fixed `issue_create` MCP tool returning "Unknown tool" after handler refactoring.
* Fixed Codex/Gemini/Copilot CLI failing on Windows with `CreateProcess error=193` — npm shim resolution now works for all CLI backends (#42).
* Fixed OpenCode CLI returning blank responses due to aggressive output filtering and premature idle timeout (#40).
* Fixed Burp AI backend returning "Unsupported backend" when used from chat.
* Fixed backend picker changes not propagating to passive/active scanners.
* Fixed CLI path resolution logging every 5 seconds via health timer.
* Fixed finding marker highlighting incorrect response regions.
* Renamed all user-facing strings from "Burp AI Agent" to "Custom AI Agent" for PortSwigger compliance.
* Fixed `ExperimentalSerializationApi` compiler warnings.
* Fixed deprecated `URL()` constructor warnings in tests.

### Added

* **Project-Scoped Chat Sessions**: Chat sessions stored in `extensionData()` (project-scoped) instead of global `preferences()`. Auto-migration from global on first open. Clear Chat fully resets messages, context, counters, drafts, tool state, and persisted data.
* **Project Change Detection**: 30-second polling of `api.project().id()`. On change: save sessions, clear memory, restore from new project, clear ScanKnowledgeBase, shutdown chat connections.
* **Output Token Limits**: All HTTP backends set `max_tokens`/`num_predict` automatically. Chat: 4096, Scanner single: 2048, Scanner batch: 4096, Adaptive payloads: 1024.
* **Scanner Project Isolation**: ScanKnowledgeBase cleared on scanner disable. PersistentPromptCache namespaced per project.

### Fixed

* **Batch analysis request loss**: flushBatch() falls back to individual analysis on failure.
* **403 bypass false positives**: Body delta check (50 bytes) on all 3 techniques.
* **CLI session ID NPE**: Safe-call replaces `!!` after CAS.
* **External backend classloader leak**: URLClassLoader closed on error.
* **Webhook crash**: try-catch for best-effort delivery.
* **AI enabled check fail-open**: `isAiEnabled()` fallback changed from `true` to `false` (fail-closed, BApp Store compliance).

## 0.4.0 (2026-03-06)

Highlights:

* **JS Endpoint Discovery** — automatic extraction of API endpoints from JavaScript responses using 8 regex patterns, with LRU dedup cache and "Extract JS Endpoints" context menu action.
* **403 Bypass Testing** — new `ACCESS_CONTROL_BYPASS` vulnerability class with IP spoofing headers, path manipulation, and HTTP method switching techniques.
* **Custom Targeted Tests** — multi-select dialog with all 55+ vulnerability classes from the Targeted Tests context menu.
* **Excluded File Extensions** — configurable passive scanner filter to skip static files (30+ default extensions).
* **Finding Request/Response Markers** — byte-range markers highlight payloads and evidence in Burp's request/response viewer.
* **Site Map Tree Node Selection** — context menu actions now work on directory/root nodes in the site map tree.
* **Keyword-Priority Response Sampling** — security-relevant excerpts from truncated response bodies.
* Improved MCP tool call prompt format for better AI tool usage accuracy.
* MCP settings optimization to skip unnecessary server restarts.
* Fixed Burp Suite shutdown hang after chat usage (#34).
* Fixed TLS certificate generation on JDK 25 (#35).
* Fixed CLI backends not visible in UI dropdown (#38).
* Removed decorative emojis from UI elements.

## 0.3.0 (2026-02-24)

Highlights:

* **Copilot CLI backend** for multi-model analysis via GitHub infrastructure.
* **AI Request Logger** with real-time activity table, trace ID correlation, preset filters, and optional rolling JSONL file persistence.
* **Auto tool chaining** — chat automatically chains up to 8 sequential MCP tool calls per interaction.
* **System prompt support** — HTTP backends receive agent instructions via the system role.
* **Per-session token tracking** with visual token bars in the sessions sidebar.
* System proxy support for HTTP backends (respects Burp/JVM proxy config).
* Improved passive scanner prompt with explicit severity definitions and DO NOT REPORT rules.
* Expanded passive scanner token/cost controls (dedup/cache/body/header/param caps).
* Manual context payload controls, compact JSON serialization, and context size cap.
* Prompt-result cache for repeated passive analyses.
* Backend health contract and richer UI health states (`OK`, `Degraded`, `Offline`).
* Active scanner queue panel and stronger backpressure coverage.
* Improved history trimming for long sessions.
* Broader integration/concurrency/resilience test coverage.
* Settings schema migration support.
* Comprehensive pre-release stability hardening (3 rounds, 29 fixes).

## 0.2.0 (2026-02-09)

Highlights:

* Chat UI overhaul with sessions persistence and export.
* Improved CLI discovery and backend availability filtering.
* Better markdown rendering and chat controls.

## 0.1.4 (2026-02-06)

Highlights:

* Backend health indicator in UI.
* Active scanner runtime stats and queue controls.
* Shared HTTP backend support and improved scanner reliability.
* Agent profile validation against MCP tool availability.

## 0.1.x

Initial feature line introducing:

* pluggable backends,
* privacy modes,
* MCP server and tools,
* passive/active scanners,
* prompt templates and Burp context actions.
