# Overview

{% embed url="https://github.com/six2dez/burp-ai-agent" %}

**AI integration for Burp Suite.**

> **A note on the name:** This extension is published as **Custom AI Agent** (formerly *Burp AI Agent*). It was renamed to comply with PortSwigger's BApp Store naming requirements and to avoid confusion with Burp Suite's built-in **Burp AI** provider. The GitHub repository (`github.com/six2dez/burp-ai-agent`), the documentation site (`burp-ai-agent.six2dez.com`), and the configuration directory (`~/.burp-ai-agent/`) keep the `burp-ai-agent` identifier for continuity.

Custom AI Agent is an extension for Burp Suite that integrates AI capabilities into your security workflow. It offers:

* **Pluggable Backends**: Use the built-in Burp AI backend (Burp Pro with _Use AI for extensions_ enabled), the native Anthropic API, local models (Ollama, LM Studio), NVIDIA NIM, Perplexity, generic OpenAI-compatible providers, or cloud CLI providers (Gemini, Claude, Codex, Copilot, OpenCode). Add custom backends via drop-in JARs. Normal chat/scanner use blocks only the **Burp AI** backend when Burp's toggle is off; independent backends work on Community. The current `ai_analyze` and `ai_passive_scan` MCP handlers have a separate unconditional Burp-AI gate, documented in the [tools reference](mcp/tools-reference.md#ai-extension-native).
* **Privacy-Aware Design**: Configurable redaction modes (Strict/Balanced/Off) default to **Balanced**. Covered cookie, credential, body, and URL carriers are sanitized before backend handoff, and a preview dialog shows the payload for auto-captured context actions. Redaction is pattern- and carrier-aware rather than a DLP guarantee; review the [documented boundaries](privacy/limitations.md#redaction-coverage-and-known-boundaries) before using a hosted backend.
* **MCP Server**: An embedded Model Context Protocol (MCP) server with 59 tools for Burp history, Repeater, Scanner, scope, and issue workflows. The full build (GitHub releases) registers all 59; the store build exposes the 8 extension-native AI tools.
* **AI Scanners**: Passive and Active scanners that analyze traffic automatically across 62 vulnerability classes. The passive scanner runs as a Burp `PassiveScanCheck` (Burp Pro).
* **Refreshed UI**: A theme-aware internal design system styles the whole settings panel and re-themes automatically when Burp switches between light and dark.
* **Curated BountyPrompt Actions**: Optional, tag-aware context menu actions loaded from JSON prompt files.
* **Custom Prompt Library**: Save free-form prompts tagged per context (HTTP request or scanner issue), managed from Settings, surfaced in a right-click **Custom prompts** submenu, with an ad-hoc editor for one-offs.
* **Audit Logging**: JSONL-based logging with per-event SHA-256 payload hashes for compliance and reproducibility.
* **AI Request Logger**: Real-time lifecycle metadata for prompts, responses, MCP calls, retries, errors, and passive-scanner operations, with trace correlation where the caller supplies it and optional rolling JSONL persistence. Prompt and response bodies are not stored here.
* **Auto Tool Chaining**: Multi-step MCP tool execution, up to 8 tool calls per interaction. Tool calls the AI emits are gated: only read-only tools with bounded output run without asking, everything else surfaces an approval card in the chat transcript first. See [MCP Security Model](mcp/security-model.md#7-tool-call-confirmation-sec-06).



<figure><img src=".gitbook/assets/Screenshot 2026-05-15 at 10.28.35.png" alt=""><figcaption></figcaption></figure>

## Key Features

| Feature | Description |
| :--- | :--- |
| **12 Built-in Backends** | Burp AI (built-in), Anthropic, Ollama, LM Studio, NVIDIA NIM, Perplexity, Generic OpenAI-compatible, Gemini CLI, Claude CLI, Codex CLI, Copilot CLI, OpenCode CLI. |
| **59 MCP Tools** | All 59 (History, Repeater, Intruder, Scanner, Scope, Site Map, Collaborator, Utilities, and more) in the full build; 8 extension-native AI tools in the store build. |
| **Auto Tool Chaining** | Up to 8 MCP tool calls chained per interaction, each subject to the SEC-06 confirmation gate — silent only for read-only, bounded-output tools. |
| **AI Request Logger** | Real-time activity log with trace ID correlation, preset filters, and optional rolling JSONL persistence. |
| **62 Vulnerability Classes** | From SQLi and XSS to cache poisoning, JWT attacks, and API security issues. |
| **3 Scan Modes** | `BUG_BOUNTY`, `PENTEST`, and `FULL` for different engagement styles. |
| **3 Privacy Modes** | `STRICT` (host anonymization on covered carriers), `BALANCED` (pragmatic, default), and `OFF` (built-in redaction disabled; custom patterns still apply). |
| **9 Prompt Templates** | Editable templates for request and issue context menu actions. |
| **Custom Prompt Library** | User-defined free-form prompts per context (HTTP request / scanner issue), with ordered menu and audit-tracked launch metadata. |
| **8 Curated BountyPrompt Actions** | Detection, recon, and advisory prompts with selective context tags. |
| **Token-Aware Controls** | Passive scanner and manual context caps, dedup windows, and prompt-result caching to reduce model spend. |
| **Burp Pro Integration** | Native `PassiveScanCheck` / `ActiveScanCheck`, Collaborator OAST, and scanner issue actions. |

## Use Cases

1. **AI-Assisted Analysis**: Analyze requests, explain JS, draft PoCs, and generate issue narratives directly from Burp context.
2. **Local Privacy**: Run local models for low-leakage workflows and keep strict redaction controls when using cloud providers.
3. **MCP Workflows**: Connect external MCP clients to Burp and run supervised tool-driven workflows.
4. **Automated Scanning**: Keep passive and active AI scanners running while you focus on manual testing.
5. **Defensible Operations**: Preserve reviewable, hash-annotated prompt bundles and stabilize covered ordering/redaction decisions with Determinism Mode. Bundles are not complete provider wire transcripts.

## Getting Started

* [**Installation**](getting-started/installation.md): Set up the extension JAR.
* [**Quick Start**](getting-started/quick-start.md): Run your first AI analysis.
* [**First Run Checklist**](getting-started/first-run-checklist.md): Validate environment and backend health.
* [**Backends**](backends/overview.md): Configure Ollama, Gemini, Claude, Codex, and OpenCode.

## Documentation

* [**User Guide**](user-guide/ui-tour.md): UI areas, context menus, sessions, and templates.
* [**BountyPrompt Actions**](user-guide/bountyprompt-actions.md): Configure and use curated BountyPrompt submenu actions.
* [**Scanners**](scanners/passive.md): Passive and Active AI scanning.
* [**MCP Reference**](mcp/overview.md): Connect external agents safely.
* [**Privacy**](privacy/privacy-modes.md): Redaction behavior and data protection boundaries.
* [**Token & Cost Management**](user-guide/token-management.md): Usage telemetry and spend control.
* [**Examples**](examples/typical-workflows.md): Typical workflows and sample prompts.
* [**Reference**](reference/settings-reference.md): Full settings, glossary, and troubleshooting.
* [**Developer**](developer/architecture.md): Architecture, data flow, and extension internals.

## Operational Controls and Boundaries

* Settings persist across restarts. The current forward-only loader migrates schemas through version 5; keep a backup before moving a profile to a newer build because there is no downgrade migration.
* Passive and active scanners enforce queue/size limits to avoid runaway resource usage.
* Outbound prompt and MCP-result paths apply the active privacy policy, with producer-specific sanitizers for structured headers and parameters. See the [known boundaries](privacy/limitations.md#redaction-coverage-and-known-boundaries).
* MCP tools are safety-gated with safe/unsafe controls and per-tool toggles.
* Session history and context size controls help limit token/cost growth.
* Audit logging provides hash-annotated JSONL records for reproducibility and correlation; authenticity still depends on external filesystem or log-retention controls.
