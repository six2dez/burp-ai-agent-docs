# Architecture

The extension is built in Kotlin targeting JVM 21, using the Burp Montoya API. It follows a layered architecture with clear module boundaries.

## Layers

```
┌─────────────────────────────────────────┐
│  1. UI Layer (Swing)                    │
│     MainTab, ChatPanel, SettingsPanel   │
├─────────────────────────────────────────┤
│  2. Context Collection                  │
│     ContextCollector (Burp selections)  │
├─────────────────────────────────────────┤
│  3. Redaction Pipeline                  │
│     Privacy modes, token/host scrubbing │
├─────────────────────────────────────────┤
│  4. Backend Adapters (Pluggable)        │
│     CLI + HTTP backends via registry    │
├─────────────────────────────────────────┤
│  5. Supervisor                          │
│     AgentSupervisor, McpSupervisor      │
├─────────────────────────────────────────┤
│  6. MCP Server                          │
│     Ktor SSE + STDIO, 53+ tools        │
├─────────────────────────────────────────┤
│  7. Scanners                            │
│     Passive AI Scanner, Active Scanner  │
├─────────────────────────────────────────┤
│  8. Audit Logging                       │
│     JSONL + SHA-256 hashing             │
├─────────────────────────────────────────┤
│  9. Alerts                              │
│     Webhook notifications               │
└─────────────────────────────────────────┘
```

## Design Goals

*   **Modularity**: Each layer can be developed, tested, and replaced independently.
*   **Testability**: Redaction is pure (no side effects). Backends are behind interfaces.
*   **Extensibility**: New backends can be added via ServiceLoader without modifying core code. New MCP tools follow a descriptor + handler pattern.
*   **Determinism**: When enabled, context ordering and redaction are stable for reproducible audit trails.
*   **Privacy-first**: Redaction is applied before any data leaves Burp, regardless of the destination.

## Key Modules

| Package | Purpose |
| :--- | :--- |
| `ui/*` | Swing UI: MainTab, ChatPanel, SettingsPanel, components (AccordionPanel, ActionCard, ToggleSwitch, PrivacyPill, DependencyBanner). |
| `ui/UiActions` | Context menu action definitions and handlers. |
| `ui/UiTheme` | Theming and color management. |
| `ui/MarkdownRenderer` | Renders AI responses as formatted Markdown in the chat panel. |
| `ui/McpHelpPanel` | MCP configuration help and quick-start instructions. |
| `context/*` | Context collection from Burp selections (requests, responses, issues). |
| `redact/*` | Redaction policies: cookie stripping, token redaction, host anonymization with salt. |
| `backends/*` | Backend registry, adapters (Ollama, LM Studio, OpenAI-compatible, Gemini, Claude, Codex, OpenCode), diagnostics. |
| `supervisor/*` | AgentSupervisor (backend lifecycle, health checks, auto-restart). |
| `mcp/*` | KtorMcpServerManager (Ktor/Netty SSE server), McpStdioBridge, McpSupervisor, McpToolCatalog, McpTools, McpToolContext, McpRequestLimiter, McpTls. |
| `scanner/*` | PassiveAiScanner, ActiveAiScanner, PayloadGenerator, ResponseAnalyzer, InjectionPointExtractor, AiScanCheck, ActiveScanModels. |
| `audit/*` | AuditLogger (JSONL), Hashing (SHA-256). |
| `alerts/*` | Webhook alerting for finding notifications. |
| `config/*` | AgentSettings (preferences storage, default values). |

## Entry Point

The extension entry point is `BurpAiAgentExtension.initialize(MontoyaApi)`, which:

1.  Initializes the backend registry (built-in + external JARs from `~/.burp-ai-agent/backends/`).
2.  Creates the audit logger, settings, supervisor, and scanners.
3.  Registers the main UI tab and context menu providers.
4.  Registers `AiScanCheck` with Burp Pro's native scanner (with Community Edition fallback).
5.  Starts the MCP server if enabled.

## Technology Stack

| Component | Technology |
| :--- | :--- |
| **Language** | Kotlin 2.1 (JVM 21) |
| **Burp API** | Montoya API 2025.12 |
| **HTTP Server** | Ktor 3.1.3 (Netty) |
| **Serialization** | kotlinx-serialization-json, Jackson |
| **HTTP Client** | OkHttp 4.12 |
| **MCP SDK** | Kotlin SDK 0.5.0 |
| **Coroutines** | kotlinx-coroutines 1.9.0 |
| **Testing** | JUnit 5, Mockito-Kotlin |
| **Build** | Gradle (Kotlin DSL), Shadow plugin for fat JAR |


## Current Architecture Notes

Key internal architecture notes:

- **UI modularization**: settings sections moved into dedicated panel classes under `ui/panels/`.
- **Shared HTTP backend layer**: `backends/http/HttpBackendSupport.kt` centralizes client/retry/history logic.
- **Scanner shared helpers**: common issue mapping/remediation/allowlist logic extracted from scanner implementations.
- **Centralized runtime constants**: `config/Defaults.kt` for scanner/backend/supervisor limits.
- **Lifecycle/resource hardening**: backend external classloader is closed on reload/shutdown.

See also: project source `docs/ARCHITECTURE.md` for the most detailed and current architecture snapshot.
