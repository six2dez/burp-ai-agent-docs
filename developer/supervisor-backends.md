# Supervisor & Backends

The supervisors coordinate backend process lifecycle, health state, and MCP server runtime.

## Agent Supervisor

The `AgentSupervisor` controls CLI and HTTP backend sessions.

### Responsibilities

* Start or attach backend sessions.
* Run health checks and expose backend state.
* Auto-restart crashed backends when policy allows it.
* Emit operational events used by diagnostics and audit flows.

### Lifecycle

```mermaid
flowchart TD
    Start[startOrAttach()] --> Alive{Backend already alive?}
    Alive -->|Yes| Reuse[Reuse existing backend session]
    Alive -->|No| Config[Build launch config from settings]
    Config --> Kind{Backend type}
    Kind -->|CLI| Cli[Spawn subprocess and bind session ID]
    Kind -->|HTTP| Http[Connect to remote endpoint and run health check]
    Cli --> Bind[Session binding]
    Http --> Bind
    Bind --> Loop[Health monitoring loop]\n2s interval
    Loop --> Healthy{Healthy?}
    Healthy -->|Yes| Loop
    Healthy -->|No| Crash[Set status: Crashed]
    Crash --> Restart{Auto-restart enabled and budget available?}
    Restart -->|Yes| Backoff[Exponential backoff and relaunch]
    Backoff --> Loop
    Restart -->|No| Stopped[Remain stopped until manual restart]
```

### Session Management

Each launch gets a unique session ID (`session-{UUID}`) so chat history, diagnostics, and backend state remain correlated.

## MCP Supervisor

`McpSupervisor` manages the MCP server independently from AI backends.

### Features

* Health checks with bounded restart attempts.
* Existing-server detection and safe handover on occupied ports.
* SSE and STDIO transport lifecycle.
* Optional TLS with local loopback trust handling.
* Explicit state transitions: `Starting -> Running/Failed -> Stopped`.

## Backend Types

### CLI Backends

* Run as subprocesses from configured commands.
* Use stdout/stderr streaming for response and diagnostics.
* Include Gemini CLI, Claude CLI, Codex CLI, Copilot CLI, OpenCode CLI.
* **Windows**: npm-installed shims are resolved automatically. The launcher detects `.cmd` siblings for shell script shims and falls back to `cmd /c` wrapping when needed.
* **Output parsing**: Each CLI backend has a dedicated output parser that strips metadata, status lines, and prompt echoes from raw stdout to extract the AI response.

### HTTP Backends

* Connect to HTTP APIs of running servers.
* Optionally auto-start local services (provider-dependent).
* Include Ollama, LM Studio, and Generic OpenAI-compatible.

## Failure Handling

* **Immediate startup exit**: marked as crashed; usually indicates config/auth/command errors.
* **Runtime crash**: restart attempts follow backoff policy.
* **Crash suppression**: repeated failures disable restart loops.
* **Manual control**: restart remains available from UI.

## Diagnostics

Backend diagnostics include:

* process exit code,
* last stdout/stderr lines,
* launch configuration details,
* retry events with attempt number, backoff delay, and failure reason (logged to [AI Request Logger](../privacy/ai-request-logger.md)).

Use Burp's extension output/errors tabs and the **AI Logger** tab for investigation.

## Trace ID Propagation

The `AgentSupervisor.send()` and `sendChat()` methods accept an optional `traceId` parameter. If not provided, a trace ID is auto-generated (`agent-turn-{UUID}` or `chat-turn-{UUID}`). This trace ID is attached to all log entries for the operation (prompt, response, error), enabling end-to-end correlation in the AI Request Logger.
