# Overview

{% embed url="https://github.com/six2dez/burp-ai-agent" %}

**AI integration for Burp Suite.**

Burp AI Agent is an extension for Burp Suite that integrates AI capabilities into your security workflow. It offers:

* **Pluggable Backends**: Use local models (Ollama, LM Studio), generic OpenAI-compatible providers, or cloud providers (Gemini, Claude, OpenAI/Codex, OpenCode). Add custom backends via drop-in JARs.
* **Privacy-First Design**: Configurable redaction modes (Strict/Balanced/Off) to scrub sensitive data before it leaves Burp.
* **MCP Server**: An embedded Model Context Protocol (MCP) server with 53+ tools for Burp history, Repeater, Scanner, scope, and issue workflows.
* **AI Scanners**: Passive and Active scanners that analyze traffic automatically across 62 vulnerability classes.
* **Curated BountyPrompt Actions**: Optional, tag-aware context menu actions loaded from JSON prompt files.
* **Audit Logging**: JSONL-based logging with SHA-256 integrity hashing for compliance and reproducibility.

<figure><img src=".gitbook/assets/overview-main.png" alt="Burp AI Agent main tab with chat and settings"><figcaption></figcaption></figure>

## Key Features

| Feature | Description |
| :--- | :--- |
| **7 Built-in Backends** | Ollama, LM Studio, Generic OpenAI-compatible, Gemini CLI, Claude CLI, Codex CLI, OpenCode CLI. |
| **53+ MCP Tools** | History, Repeater, Intruder, Scanner, Scope, Site Map, Collaborator, Utilities, and more. |
| **62 Vulnerability Classes** | From SQLi and XSS to cache poisoning, JWT attacks, and API security issues. |
| **3 Scan Modes** | `BUG_BOUNTY`, `PENTEST`, and `FULL` for different engagement styles. |
| **3 Privacy Modes** | `STRICT` (zero trust), `BALANCED` (pragmatic), and `OFF` (raw data). |
| **9 Prompt Templates** | Editable templates for request and issue context menu actions. |
| **8 Curated BountyPrompt Actions** | Detection, recon, and advisory prompts with selective context tags. |
| **Token-Aware Controls** | Passive scanner and manual context caps, dedup windows, and prompt-result caching to reduce model spend. |
| **Burp Pro Integration** | Native `ScanCheck`, Collaborator OAST, and scanner issue actions. |

## Use Cases

1. **AI-Assisted Analysis**: Analyze requests, explain JS, draft PoCs, and generate issue narratives directly from Burp context.
2. **Local Privacy**: Run local models for low-leakage workflows and keep strict redaction controls when using cloud providers.
3. **MCP Workflows**: Connect external MCP clients to Burp and run supervised tool-driven workflows.
4. **Automated Scanning**: Keep passive and active AI scanners running while you focus on manual testing.
5. **Defensible Operations**: Preserve auditable, reproducible prompt bundles with deterministic redaction options.

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

## Operational Guarantees

* Your settings persist across restarts and are migrated safely between versions.
* Passive and active scanners enforce queue/size limits to avoid runaway resource usage.
* Privacy policies are applied before prompt data leaves Burp.
* MCP tools are safety-gated with safe/unsafe controls and per-tool toggles.
* Session history and context size controls help limit token/cost growth.
* Audit logging provides tamper-evident JSONL records for reproducibility workflows.
