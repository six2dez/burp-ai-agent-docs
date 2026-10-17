# Backends Overview

Custom AI Agent is backend-agnostic. You can run the built-in Burp AI backend, local models, OpenAI-compatible and other HTTP providers, or (in the GitHub full build) cloud CLI providers. The BApp Store build ships 7 backends: Burp AI plus the 6 HTTP backends. The GitHub full build ships 12: the same 7 plus the 5 CLI backends, and it also loads custom backends dropped in as JARs. See [Which Build?](../getting-started/which-build.md).

## Backend Selection Guide

```mermaid
flowchart TD
    Start[Choose backend]
    BurpPro{"Burp AI usable, with Burp's Use AI enabled for the extension?"}
    Privacy{Need maximum privacy?}
    Cli{"Prefer a CLI provider workflow (GitHub full build only)?"}
    OwnApi{Using your own API/provider endpoint?}

    Start --> BurpPro
    BurpPro -->|Yes| BurpAi[Burp AI built-in]
    BurpPro -->|No| Privacy

    Privacy -->|Yes| Local["Local backends<br/>Ollama or LM Studio"]
    Privacy -->|No| Cli

    Cli -->|Yes| CloudCli[Gemini CLI / Claude CLI / Codex CLI / Copilot CLI / OpenCode CLI]
    Cli -->|No| OwnApi

    OwnApi -->|Yes| Generic[Generic OpenAI-compatible]
    OwnApi -->|No| Local
```

## Supported Backends

| Backend | Type | Privacy Posture | Typical Use |
| :--- | :--- | :--- | :--- |
| **Burp AI (built-in)** | In-process (Burp's AI) | High (no extra outbound) | Burp users with AI credits and Burp's "Use AI" enabled for the extension (PortSwigger documents it as a per-extension checkbox, off by default: **Extensions > Installed > Custom AI Agent > Use AI**). |
| **Ollama** | Local HTTP | High | Offline or strict data control. |
| **LM Studio** | Local HTTP | High | Local models with GUI management. |
| **NVIDIA NIM** | Cloud HTTP | Medium | NVIDIA-hosted models (e.g. `moonshotai/kimi-k2.5`) via `integrate.api.nvidia.com`. |
| **Perplexity** | Cloud HTTP | Medium | Sonar family of web-aware reasoning models via `api.perplexity.ai`. |
| **Anthropic** | Cloud HTTP | Medium | Native Anthropic Messages API (Claude) via `api.anthropic.com`; encrypted key + token counting (v0.9.0). |
| **Generic (OpenAI-compatible)** | HTTP | Medium | Any compatible provider endpoint. |
| **Gemini CLI** | Cloud CLI (GitHub full build only) | Medium | Large-context cloud workflows. |
| **Claude CLI** | Cloud CLI (GitHub full build only) | Medium | Reasoning-heavy analysis. |
| **Codex CLI** | Cloud CLI (GitHub full build only) | Medium | Code/security analysis and PoCs. |
| **Copilot CLI** | Cloud CLI (GitHub full build only) | Medium | Multi-model analysis via GitHub infrastructure. |
| **OpenCode CLI** | Cloud CLI (GitHub full build only) | Medium | Multi-provider via one CLI. |

{% hint style="info" %}
**Network transport:** normal chat and scanner requests from every built-in HTTP backend use Burp's `MontoyaHttpTransport`, so those requests respect Burp's upstream proxy/TLS configuration and are visible in Proxy history. Health checks go through the same transport: Ollama, LM Studio and Generic OpenAI-compatible send a `GET` to the server, and NVIDIA NIM and Perplexity send a free `GET /v1/models`, which carries no captured Burp context and runs no billed completion. Anthropic performs no live health request; a non-empty key is treated as healthy until an actual request proves otherwise. Every one of these requests, health checks included, passes the "Use AI" gate described below.
{% endhint %}

### Capability Matrix

| Backend | Response delivery in current runtime | JSON mode | System role | Auto-start |
| :--- | :-: | :-: | :-: | :-: |
| **Burp AI (built-in)** | No (single `execute`) | No — enforced via prompt | Yes | N/A |
| **Ollama** | Single chunk (`stream=false`) | Yes (`format=json`) | Yes | Yes (`ollama serve`) |
| **LM Studio** | Single chunk (`stream=false`) | Yes (`response_format=json_object`) | Yes | Yes (`lms server start`) |
| **NVIDIA NIM** | Single parsed response; see caveat below | Yes (`response_format=json_object`) | Yes | N/A |
| **Perplexity** | Single parsed response; see caveat below | **No** (Sonar API rejects `response_format=json_object`) | Yes | N/A |
| **Anthropic** | Single chunk (`stream=false`, proxy-visible) | No — enforced via prompt | Yes (native `system`) | N/A |
| **Generic OpenAI-compatible** | Single chunk (`stream=false`) | Yes (`response_format=json_object`) | Yes | N/A |
| **Gemini CLI** | Buffered process output, then one chunk | No | No (prepended) | N/A |
| **Claude CLI** | Buffered process output, then one chunk | No | No (prepended) | N/A |
| **Codex CLI** | Buffered process/output file, then one chunk | No | No (prepended) | N/A |
| **Copilot CLI** | Buffered process output, then one chunk | No | No (prepended) | N/A |
| **OpenCode CLI** | Buffered process output, then one chunk | No | No (prepended) | N/A |

The `AgentConnection` API supports repeated `onChunk` callbacks, but the current built-ins call it once after a complete response has been buffered and parsed. NVIDIA NIM and Perplexity currently set `stream: true` in their normal chat-completions payload while the shared Montoya path parses the returned body as one JSON document; a provider that returns actual SSE `data:` frames is not decoded by that path. Treat this as a current compatibility limitation, not working UI streaming.

See [Agent Profiles → How It Works](../user-guide/agent-profiles.md#how-it-works) for how the system-role difference affects profile delivery.

## Setup Path

1. Open the **AI Backend** tab in Settings.
2. Select **Preferred Backend** for new sessions.
3. Configure command/URL/model/auth fields for that backend.
4. Use **Test connection** where available.
5. Start with [Privacy Modes](../privacy/privacy-modes.md) set appropriately.

{% tabs %}
{% tab title="CLI Backends" %}
GitHub full build only.

Configure executable command and ensure authentication is already completed in the same runtime environment as Burp.

Windows tip: with npm-installed tools, prefer full shim paths like `C:\\Users\\<you>\\AppData\\Roaming\\npm\\claude.cmd`.
{% endtab %}

{% tab title="HTTP Backends" %}
Configure base URL, model, optional API key, and extra headers.

For local servers, verify the service is running and port is reachable from Burp.
{% endtab %}

{% tab title="Custom Drop-in" %}
GitHub full build only: the BApp Store build never creates, lists or loads this directory.

Drop custom backend JARs implementing `AiBackendFactory` into:

`~/.burp-ai-agent/backends/`

Restart Burp to load them.
{% endtab %}
{% endtabs %}

## Cross-Platform CLI Detection

The CLI backends are in the GitHub full build only. They depend on environment inheritance from the Burp process.

* If Burp starts from GUI, shell `PATH` and env vars may differ.
* Use explicit command paths when detection fails.
* For Windows + WSL bridge patterns, see backend-specific pages and [Troubleshooting](../reference/troubleshooting.md).

### Windows npm Shim Resolution

On Windows, npm-installed CLI tools (Codex, Gemini, OpenCode, Copilot) install as shell script shims that Java cannot execute directly. The extension automatically resolves these:

1. **`.cmd` sibling detection**: If the resolved path points to a non-executable shim, the resolver looks for a `.cmd` sibling (e.g., `codex` -> `codex.cmd`).
2. **npm directory scanning**: Checks `%APPDATA%\npm`, `%LOCALAPPDATA%\npm`, and `%USERPROFILE%\AppData\Roaming\npm` for `.cmd` shims.
3. **Fallback wrapper**: If no `.cmd` sibling is found, wraps the command with `cmd /c`.

This eliminates the `CreateProcess error=193` that occurs when Java tries to execute shell script shims directly.

## Burp Edition Notes and the "Use AI" Gate

**BApp Store build.** The BApp Store build runs on Burp Community and Burp Professional; it sends AI requests only while Burp's "Use AI" setting is enabled for the extension, and with it off it loads and refuses every AI request. That covers every backend on this page, Burp AI included: while "Use AI" is off, the status pill reads "AI disabled in Burp" whatever backend is selected.

**GitHub full build.** Every backend except **Burp AI (built-in)** works whatever "Use AI" says, including the CLI backends. Burp AI needs "Use AI" in both builds.

In both builds, MCP `ai_analyze` and `ai_passive_scan` return a tool error while "Use AI" is off. Which Burp editions can turn "Use AI" on is decided by Burp, not by this extension. PortSwigger documents "Use AI" as a per-extension checkbox, off by default: **Extensions > Installed > Custom AI Agent > Use AI**. Some MCP tools of the GitHub full build (the scanner tools) also need Burp Professional; see the [MCP Tools Reference](../mcp/tools-reference.md). See [Which Build?](../getting-started/which-build.md#burp-edition-requirements).

## Health States

The selected backend's health is shown as a colored pill in the top bar. When the checks run depends on the build:

* **BApp Store build:** on demand only. A check runs after **Save** in Settings, after a backend change and when you click the pill, and the first chat reply repaints it. Nothing is checked at startup or on a timer, so the pill reads "Not checked · click to test" until then.
* **GitHub full build:** a startup check for every backend, then every 30 seconds for local backends only (Burp AI, the CLI backends, and HTTP servers on a loopback address). Remote providers are checked again only after a settings or backend change and when you click the pill.

In both builds a chat reply also repaints the pill, with no extra request, and **Test connection** in Settings runs a check of its own and shows the result in a dialog. Before any check is sent, the pill reads Burp's "Use AI" setting locally: if the selected backend is refused, it shows "AI disabled in Burp" and sends nothing. That local read repeats about once a second, so turning "Use AI" on or off shows up without a click.

| Pill | Internal state | Meaning | Typical cause |
| :--- | :--- | :--- | :--- |
| **Not checked · click to test** | none yet | BApp Store build, before the first check. | Nothing has asked for a check yet. |
| **AI: ?** | none yet | GitHub full build, before the startup check. | The startup check has not run yet. |
| **AI: OK** (green) | `Healthy` | The backend-specific check passed. This can mean HTTP reachability, an executable CLI command, Burp AI enabled, or—for Anthropic—only that a key is present. It does not guarantee the next inference will succeed. | Normal configured state. |
| **AI: Degraded** (amber) | `Degraded` | An HTTP health `GET` reached the endpoint but received HTTP 401, 403 or 429. The tooltip carries the status. | Invalid/expired API key or rate limiting. |
| **AI: Offline** (red) | `Unavailable` | The configured check failed or required configuration is absent. | Missing URL/model, unreachable endpoint, or unresolved CLI executable. |
| **AI disabled in Burp** | refused by the gate | Burp's "Use AI" is off and the build refuses the selected backend: any backend in the BApp Store build, Burp AI in the GitHub full build. The tooltip carries the refusal text. No request is sent. | "Use AI" is off for the extension. |

The check runs off the Swing event thread. Status transitions themselves are not written to the AI Request Logger; actual prompt failures and transport retries are.

If a backend stays `Offline` longer than expected, see [Backend Troubleshooting](troubleshooting.md) for per-backend error signatures.

## Retry Behavior

HTTP backends (Ollama, LM Studio, OpenAI-compatible, NVIDIA NIM, Perplexity, Anthropic) include automatic retry logic with bounded stepped backoff:

* **Maximum attempts**: 6 total (the initial attempt plus at most 5 retries).
* **Retryable errors**: Transport exceptions classified as transient, including connection timeout/refused, unexpected end of stream, stream reset, and end-of-input conditions. HTTP error responses are returned immediately rather than retried inside the same call.
* **Backoff before the five retries**: `500 ms`, `1000 ms`, `1500 ms`, `2000 ms`, `3000 ms`.
* **Diagnostics**: Each retry attempt is logged to the [AI Request Logger](../privacy/ai-request-logger.md) as a `RETRY` activity with the attempt number, delay, and reason.

### Circuit Breaker

Each launched HTTP connection has its own circuit breaker:

* **Failure threshold**: 5 recorded transient failures open the circuit. Transport exceptions count per failed attempt; HTTP 429 and 5xx responses count once, while other 4xx responses do not.
* **Reset timeout**: 30 seconds before the next model request can become the one half-open attempt.
* **Half-open request**: success closes the breaker; failure reopens it.
* When the circuit is open the backend fails fast with `"<backend> backend is temporarily unavailable (circuit open)"`.

The top-bar health checker is independent of this per-connection breaker. Starting a new chat session creates a new connection; it does not mutate the breaker held by an existing session.

The **Burp AI (built-in)** backend uses Burp's own retry and error handling, so the schedule above does not apply to it. CLI backends (GitHub full build only) have no per-request retry schedule.

CLI processes also have a shared **120-second hard runtime ceiling**. OpenCode adds a separate 30-second idle-output timeout; either limit can terminate its run first. The current settings panel does not expose the shared CLI timeout as an editable field.

## Output Token Limits

HTTP backends (Ollama, LM Studio, OpenAI-compatible, NVIDIA NIM, Perplexity, Anthropic) pass output-token limits per request type to bound response size. A limit cannot guarantee that a model completes the requested answer:

| Request Type | Max Output Tokens |
| :--- | :--- |
| **Chat** | 4096 |
| **Scanner (single request)** | 2048 |
| **Scanner (batch analysis)** | 4096 |
| **Payload generation** | 1024 |

CLI backends manage their own output limits through their respective configurations and are not subject to these values.

## Next Steps

* [Burp AI (Built-in)](burp-ai.md)
* [Ollama (Local)](ollama.md)
* [LM Studio (Local)](lm-studio.md)
* [NVIDIA NIM](nvidia-nim.md)
* [Perplexity](perplexity.md)
* [Anthropic (API)](anthropic.md)
* [Generic (OpenAI-compatible)](openai-compatible.md)
* [Gemini CLI](gemini-cli.md)
* [Claude CLI](claude-cli.md)
* [Codex CLI](codex-cli.md)
* [Copilot CLI](copilot-cli.md)
* [OpenCode CLI](opencode-cli.md)
