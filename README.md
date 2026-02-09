# Overview

{% embed url="https://github.com/six2dez/burp-ai-agent" %}

**The bridge between Burp Suite and modern AI.**

Burp AI Agent is an extension for Burp Suite that integrates AI capabilities into your security workflow. It offers:

* **Pluggable Backends**: Use local models (Ollama, LM Studio), generic OpenAI-compatible providers, or cloud providers (Gemini, Claude, OpenAI/Codex, OpenCode). Add custom backends via drop-in JARs.
* **Privacy-First Design**: Configurable redaction modes (Strict/Balanced/Off) to scrub sensitive data _before_ it leaves Burp.
* **MCP Server**: A full Model Context Protocol (MCP) server with 53+ tools that allows external AI agents (like Claude Desktop) to interact with Burp — browsing history, sending requests, scanning, and creating issues autonomously.
* **AI Scanners**: Passive and Active scanners that autonomously analyze traffic and flag vulnerabilities across 62 vulnerability classes.
* **Audit Logging**: JSONL-based logging with SHA-256 integrity hashing for compliance and reproducibility.

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Key Features

| Feature                      | Description                                                                                    |
| ---------------------------- | ---------------------------------------------------------------------------------------------- |
| **7 Built-in Backends**      | Ollama, LM Studio, Generic OpenAI-compatible, Gemini CLI, Claude CLI, Codex CLI, OpenCode CLI. |
| **53+ MCP Tools**            | History, Repeater, Intruder, Scanner, Scope, Site Map, Collaborator, Utilities, and more.      |
| **62 Vulnerability Classes** | From SQLi and XSS to cache poisoning, JWT attacks, and API security issues.                    |
| **3 Scan Modes**             | BUG\_BOUNTY, PENTEST, and FULL for different engagement types.                                 |
| **3 Privacy Modes**          | STRICT (zero trust), BALANCED (pragmatic), OFF (raw data).                                     |
| **9 Prompt Templates**       | Customizable templates for all context menu actions.                                           |
| **Burp Pro Integration**     | Native ScanCheck, Collaborator OAST, scanner issue actions.                                    |

## Why use this?

1. **AI-Assisted Analysis**: Use AI to analyze requests, explain JS, or draft proof-of-concept (PoC) exploits.
2. **Local Privacy**: Run Llama 3 or Mistral locally via Ollama for zero-data-leakage analysis.
3. **Agentic Workflows**: Connect Claude Desktop to Burp via MCP to let the AI "drive" Burp — navigating the site map, sending test requests, and verifying issues autonomously.
4. **Automated Scanning**: Let passive and active AI scanners work in the background while you focus on manual testing.
5. **Compliance Ready**: Audit logging with SHA-256 hashing, deterministic redaction, and configurable privacy modes.

## Getting Started

* [**Installation**](getting-started/installation.md): Set up the JAR.
* [**Quick Start**](getting-started/quick-start.md): Your first AI analysis in 5 minutes.
* [**First Run Checklist**](getting-started/first-run-checklist.md): Verify your setup.
* [**Backends**](backends/overview.md): Configure Ollama, Gemini, Claude, and more.

## Documentation

* [**User Guide**](user-guide/ui-tour.md): Tour the UI, context menus, chat, and prompt templates.
* [**Scanners**](scanners/passive.md): Passive and Active AI scanning.
* [**MCP Reference**](mcp/overview.md): Learn how to connect external agents.
* [**Privacy**](privacy/privacy-modes.md): Understand how your data is protected.
* [**Examples**](examples/typical-workflows.md): Real-world workflows and sample prompts.
* [**Reference**](reference/settings-reference.md): Complete settings, glossary, and troubleshooting.
* [**Developer**](developer/architecture.md): Architecture, data flow, and extension development.

## Operational Notes

* Passive scanner reliability hardening (atomic state, bounded backend startup polling, latch-based completion wait).
* Active scanner queue backpressure with a hard cap (default: 2000 queued targets) to prevent unbounded growth.
* Shared HTTP backend support for retry/client/history logic across Ollama, LM Studio, and OpenAI-compatible providers.
* Type-safe scanner settings in code (`SeverityLevel`, `PayloadRisk`, `ScanMode`) with backward-compatible persistence.
* AGENTS profile validation against enabled MCP tools, with warnings in Settings.
* Settings UI modularization into dedicated panel classes (Backend, Passive, Active, MCP, Prompt, Privacy, Help).
* Passive scanner issue details now include model reasoning when provided by backend JSON output.
* Structured prompt defaults moved to role/task/scope/output format for better response consistency.
* Additional unit tests for scanner parsing/generation/analyzers and backend conversation history.
