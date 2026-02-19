# Architecture

The extension is built in Kotlin on the JVM using Burp Montoya API. It follows layered boundaries so UI, context, privacy, backend, supervision, scanner, MCP, and audit concerns remain separable.

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
│  4. Prompt Resolution                   │
│     Built-in templates + BountyPrompt   │
├─────────────────────────────────────────┤
│  5. Backend Adapters (Pluggable)        │
│     CLI + HTTP backends via registry    │
├─────────────────────────────────────────┤
│  6. Supervisor                          │
│     AgentSupervisor, McpSupervisor      │
├─────────────────────────────────────────┤
│  7. MCP Server                          │
│     SSE + STDIO tools gateway           │
├─────────────────────────────────────────┤
│  8. Scanners                            │
│     Passive AI Scanner, Active Scanner  │
├─────────────────────────────────────────┤
│  9. Audit Logging                       │
│     JSONL + SHA-256 hashing             │
├─────────────────────────────────────────┤
│ 10. Alerts                              │
│     Webhook notifications               │
└─────────────────────────────────────────┘
```

## Design Goals

* **Modularity**: Layers can evolve independently.
* **Testability**: Redaction and parsing logic remain unit-testable.
* **Extensibility**: Backends and tools are pluggable.
* **Determinism**: Stable context ordering and stable anonymization when enabled.
* **Privacy-first**: Redaction runs before outbound backend calls.

## Key Modules

| Package | Purpose |
| :--- | :--- |
| `ui/*` | Swing UI, settings panels, and interaction components. |
| `ui/UiActions` | Context menu wiring for request/response and issue actions. |
| `context/*` | Context collection from Burp selections. |
| `redact/*` | Privacy policy and redaction engine. |
| `prompts/bountyprompt/*` | Curated BountyPrompt catalog, loader, tag resolver, and output parser. |
| `backends/*` | Backend registry, built-in adapters, diagnostics. |
| `supervisor/*` | Backend and MCP lifecycle supervisors. |
| `mcp/*` | MCP server manager, tool catalog, request limiter, and transport bridges. |
| `scanner/*` | Passive and active scanner engines plus analyzers/generators. |
| `audit/*` | JSONL audit writer and hash utilities. |
| `alerts/*` | Optional webhook notifications. |
| `config/*` | Settings model, persistence, and defaults. |

## Entry Point

`BurpAiAgentExtension.initialize(MontoyaApi)` performs:

1. Backend registry initialization (built-ins plus drop-in JARs).
2. Settings/audit/supervisor/scanner initialization.
3. UI tab and context menu provider registration.
4. Scanner integration registration (Burp Pro path plus Community fallback).
5. MCP startup when enabled.

## BountyPrompt Integration Path

BountyPrompt actions are integrated through `UiActions` and `prompts/bountyprompt/*`:

1. Loader reads curated JSON prompts from configured directory.
2. Resolver applies redaction-aware tag substitution.
3. Composed prompt is sent through normal chat backend pipeline.
4. Optional output parsing creates Burp issues through confidence gating.
5. Audit events record action invocation and issue creation results.

## Core Technology Components

| Component | Technology |
| :--- | :--- |
| **Language** | Kotlin (JVM) |
| **Burp API** | Montoya API |
| **HTTP Server** | Ktor (Netty) |
| **Serialization** | kotlinx-serialization, Jackson |
| **HTTP Client** | OkHttp |
| **Coroutines** | kotlinx-coroutines |
| **Testing** | JUnit, Mockito-Kotlin |
| **Build** | Gradle Kotlin DSL with Shadow JAR |

## Operational Guarantees

* Settings UI is split into dedicated tab/panel classes.
* Shared HTTP backend support centralizes retry/client/history behavior.
* Scanner helpers centralize shared mapping/remediation logic.
* Runtime limits are centralized in defaults/constants.
* External backend classloaders are closed during reload/shutdown.
