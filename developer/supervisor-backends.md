# Supervisor & Backends

The Supervisor is the internal component responsible for managing the lifecycle of AI backend processes and the MCP server.

## Agent Supervisor

The `AgentSupervisor` manages CLI and HTTP backend connections.

### Responsibilities

*   **Start or attach** to a backend for a new session.
*   **Monitor health** via periodic checks (every 2 seconds).
*   **Auto-restart** on failure with exponential backoff.
*   **Track crash count** and suppress restarts after repeated failures.
*   **Expose status** to the UI (Running, Crashed, Stopped).
*   **Emit audit events** for session start/stop.

### Lifecycle

```
startOrAttach()
    │
    ├── Backend already alive? ──► Reuse existing connection
    │
    └── Build launch config from settings
            │
            ▼
        Launch backend
            │
            ├── CLI: Spawn subprocess with env vars
            │       └── Generate session ID: "session-{UUID}"
            │
            └── HTTP: Connect to running server
                    └── Verify with health check
            │
            ▼
        Session binding (session ID ↔ backend ID)
            │
            ▼
        Health monitoring loop (2s interval)
            │
            ├── Process alive? ──► Continue monitoring
            │
            └── Process exited? ──► Status = "Crashed"
                    │
                    ├── Auto-restart enabled? ──► Re-launch with backoff
                    │
                    └── Auto-restart disabled? ──► Stay stopped
```

### Session Management

Each backend launch generates a unique session ID (`session-{UUID}`). This ID:
*   Ties the chat session to a specific backend instance.
*   Appears in audit logs for traceability.
*   Allows multiple backends to run in parallel across different sessions.

## MCP Supervisor

The `McpSupervisor` manages the MCP server lifecycle separately from the AI backend.

### Features

*   **Health checks** with automatic restart (up to 4 retries).
*   **Existing server detection**: If port is occupied, requests shutdown before starting.
*   **Dual transport support**: HTTP (SSE via Ktor/Netty) and STDIO bridge.
*   **TLS support**: Loopback trust bypass for local connections.
*   **State machine**: Starting → Running/Failed → Stopped.

## Backend Types

### CLI Backends
*   Spawned as subprocesses via the configured shell command.
*   Output collected from process stdout/stderr.
*   Environment variables injected for backend communication.
*   Supported: Gemini CLI, Claude CLI, Codex CLI, OpenCode CLI.

### HTTP Backends
*   Direct HTTP API calls to a running server.
*   No subprocess management needed.
*   Optional auto-start if the server isn't running.
*   Supported: Ollama, LM Studio, Generic (OpenAI-compatible).

## Launch Modes

The supervisor runs CLI backend processes internally (embedded mode). Output streams directly to the extension's chat panel.

## Failure Handling

*   **Immediate exit**: If a CLI backend exits within seconds of launch, the supervisor marks it as "Crashed" and does **not** auto-restart (likely a configuration error).
*   **Runtime crash**: If a backend crashes after running successfully, auto-restart is attempted with exponential backoff.
*   **Crash suppression**: After multiple consecutive crashes, auto-restart is disabled to prevent restart loops.
*   **Manual restart**: The user can always restart a backend manually from the UI.

## Backend Diagnostics

The `BackendDiagnostics` module logs diagnostic information when backends fail:
*   Exit code from the process.
*   Last lines of stdout/stderr output.
*   Environment variables used for launch.

This information is available in the Burp extension output tab.
