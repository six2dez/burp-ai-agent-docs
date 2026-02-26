# Changelog (User-Facing Summary)

This page summarizes notable release changes for operators and testers.

For full technical detail, see the project changelog in the plugin repository.

## 0.3.0 (2026-02-24)

Highlights:

* Expanded passive scanner token/cost controls (dedup/cache/body/header/param caps).
* Manual context payload controls and compact JSON serialization.
* Prompt-result cache for repeated passive analyses.
* Backend health contract and richer UI health states (`OK`, `Degraded`, `Offline`).
* Active scanner queue panel and stronger backpressure coverage.
* Improved history trimming for long sessions.
* Broader integration/concurrency/resilience test coverage.
* Settings schema migration support.

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
