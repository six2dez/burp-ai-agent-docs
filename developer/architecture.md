# Architecture

Burp AI Agent is implemented in Kotlin on the JVM using the Burp Montoya API. The architecture is intentionally layered so UI, context collection, redaction, backend execution, scanning, MCP, and audit concerns can evolve independently.

## Layered Design

```mermaid
flowchart TD
    subgraph L1[1. UI Layer]
      UI[MainTab, ChatPanel, SettingsPanel]
    end

    subgraph L2[2. Context Collection]
      CC[ContextCollector]
    end

    subgraph L3[3. Redaction Pipeline]
      RP[Privacy modes and sanitization]
    end

    subgraph L4[4. Prompt Resolution]
      PR[Built-in templates and BountyPrompt]
    end

    subgraph L5[5. Backend Adapters]
      BA[CLI and HTTP backends]
    end

    subgraph L6[6. Supervisors]
      SUP[AgentSupervisor and McpSupervisor]
    end

    subgraph L7[7. MCP Server]
      MCP[SSE and STDIO bridge]
    end

    subgraph L8[8. Scanners]
      SCN[Passive and Active scanners]
    end

    subgraph L9[9. Audit Logging]
      AUD[JSONL events and hashes]
    end

    subgraph L10[10. Alerts]
      ALT[Optional webhook notifications]
    end

    UI --> CC --> RP --> PR --> BA
    BA --> SUP
    CC --> SCN
    RP --> MCP
    BA --> AUD
    SCN --> AUD
    MCP --> AUD
    AUD --> ALT
```

## Initialization Sequence

`BurpAiAgentExtension.initialize(MontoyaApi)` performs startup in a strict order.

```mermaid
sequenceDiagram
    participant Burp as Burp Suite
    participant Ext as BurpAiAgentExtension
    participant Reg as BackendRegistry
    participant Core as Settings/Supervisors/Scanners
    participant UI as MainTab + Menus
    participant MCP as MCP Server

    Burp->>Ext: initialize(api)
    Ext->>Reg: load built-in + drop-in backends
    Ext->>Core: initialize settings, audit, scanners, supervisors
    Ext->>UI: register tab and context menus
    Ext->>Core: register scanner integration (Pro/Community paths)
    alt MCP enabled
      Ext->>MCP: start server
    else MCP disabled
      Ext-->>Burp: continue without MCP transport
    end
```

## Design Goals

* **Modularity**: separate concerns to reduce coupling.
* **Testability**: keep parsing and privacy logic unit-testable.
* **Extensibility**: backend/tool additions should not require core refactors.
* **Determinism**: stable ordering and stable anonymization when configured.
* **Privacy-first**: redaction happens before outbound backend or MCP output.

## Key Modules

| Package | Purpose |
| :--- | :--- |
| `ui/*` | Swing UI, settings panels, interaction components. |
| `ui/UiActions` | Context menu wiring for request/response and issue actions. |
| `context/*` | Context collection from Burp selections. |
| `redact/*` | Privacy policy and redaction engine. |
| `prompts/bountyprompt/*` | Curated prompt loader, resolver, parser, and catalog. |
| `backends/*` | Backend registry, built-in adapters, diagnostics. |
| `supervisor/*` | Backend and MCP lifecycle supervision. |
| `mcp/*` | MCP manager, catalog, limiter, and transports. |
| `scanner/*` | Passive/active scanner engines and analyzers. |
| `audit/*` | JSONL writer and integrity hashes. |
| `alerts/*` | Optional webhook notifications. |
| `config/*` | Settings model, persistence, defaults, migration. |

## Related Pages

* [Data Flow](data-flow.md)
* [Redaction Pipeline](redaction-pipeline.md)
* [Supervisor & Backends](supervisor-backends.md)
* [Feature Coverage (0.3.0)](feature-coverage-0.3.md)
