# MCP Overview

Model Context Protocol (MCP) lets external AI clients use Burp data and actions through a controlled interface.

## What MCP Enables

With MCP enabled, an external AI client can:

* query Burp history,
* run analysis tools,
* send controlled requests,
* create issues programmatically.

This keeps the operator in control while expanding automation options.

## Connection Topology

{% tabs %}
{% tab title="SSE (Default)" %}
Primary transport for MCP clients.

Default endpoint:

```
http://127.0.0.1:9876/sse
```

For external access, enable TLS and include `Authorization: Bearer <token>`.
{% endtab %}

{% tab title="STDIO Bridge" %}
Optional transport for process-based clients.

Enable **STDIO Bridge** in the **MCP Server** settings tab.

Useful when your client expects stdin/stdout MCP server behavior.
{% endtab %}
{% endtabs %}

## Cloud Client Setup (SSE via stdio bridge)

Some desktop clients expect a stdio MCP process. `supergateway` bridges to Burp SSE.

The `burp-ai-agent` key below is just the name your client uses to identify the server — you can rename it freely. The internal `Implementation` string advertised during the MCP handshake is also `burp-ai-agent`, for historical compatibility.

```json
{
  "mcpServers": {
    "burp-ai-agent": {
      "command": "npx",
      "args": [
        "-y",
        "supergateway",
        "--sse",
        "http://127.0.0.1:9876/sse"
      ]
    }
  }
}
```

If token is required:

```json
{
  "mcpServers": {
    "burp-ai-agent": {
      "command": "npx",
      "args": [
        "-y",
        "supergateway",
        "--sse",
        "http://127.0.0.1:9876/sse",
        "--oauth2Bearer",
        "your-token"
      ]
    }
  }
}
```

## Build Variants and Tool Exposure

The extension ships two build artifacts, and the build you load determines which MCP tools are registered:

* **BApp Store build** (`./gradlew shadowJar -PstoreBuild=true` → `Custom-AI-Agent-0.8.0.jar`): registers **only the 8 extension-native AI tools** — `status`, `issue_create`, `ai_analyze`, `ai_passive_scan`, `ai_findings_recent`, `redact_preview`, `ai_audit_query`, `ai_backends_list`. Generic Burp/Montoya tools (proxy history, repeater, scanner, scope, site map, intruder, Collaborator, utilities, etc.) are intentionally **not** exposed here. For those, use PortSwigger's official Burp MCP Server alongside this extension.
* **Full build** (`./gradlew shadowJar` → `Custom-AI-Agent-full-0.8.0.jar`, GitHub releases): registers **all 59 MCP tools**, including the generic Montoya tools above.

A compile-time `BuildFlags.STORE_BUILD` constant gates which tools register. The AI-calling tools (`ai_analyze`, `ai_passive_scan`, …) also check `ai.isEnabled()` before issuing a request, so the configured AI setting is respected; independent third-party backends still work when Burp's built-in AI is off.

When loaded, the extension appears in Burp's **Extensions** list and as a Suite tab titled **Custom AI Agent** (named that way to distinguish it from Burp's built-in "Burp AI" provider).

## Features

* SSE and optional STDIO transport.
* 59 MCP tools across Burp workflows in the full build (8 extension-native AI tools in the BApp Store build); see [Tools Reference](tools-reference.md).
* Unsafe-tool gating with per-tool toggles.
* Optional **Restrict MCP tools to in-scope hosts** (`mcpScopeOnly`) that confines every scope-aware tool to Burp's defined scope.
* Configurable request limiter and body-size caps.
* Proxy-history preprocessing pipeline (binary filter, size cap, content-type allowlist, newest-first, raw opt-in). See [MCP Proxy History Preprocessing](../reference/settings-reference.md#mcp-proxy-history-preprocessing).
* Administrative endpoints (`GET /__mcp/health`, `POST /__mcp/shutdown`) used for health checks and safe takeover.
* Auto-restart of the MCP listener on unexpected termination.
* Privacy-aware tool output filtering.
* Inline advisory banner in the **MCP Server** settings tab that surfaces risky combinations (external access without allowed origins, external access with **Enable Unsafe Tools** on, etc.). See [UI Tour → Advisory Banner (SubtleNotice)](../user-guide/ui-tour.md#advisory-banner-subtlenotice).

<figure><img src="../.gitbook/assets/Screenshot 2026-05-15 at 10.33.38.png" alt=""><figcaption></figcaption></figure>

## Administrative Endpoints

| Endpoint          | Method | Auth                            | Purpose                                                                                                                                                                                   |
| ----------------- | ------ | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/__mcp/health`   | GET    | None — loopback only            | Returns `"ok"` with the marker header `X-Burp-AI-Agent: mcp`. Used by the MCP Supervisor to detect a pre-existing Custom AI Agent listener on the same port before attempting a takeover. |
| `/__mcp/shutdown` | POST   | `Authorization: Bearer <token>` | Used during port takeover: a new MCP server instance sends this to ask a colliding older instance to release the port. Rejects requests without a valid bearer.                           |

## Port Conflict Handling and Takeover

When the MCP Supervisor starts a server and the port is already in use:

1. It probes `GET <scheme>://<host>:<port>/__mcp/health` with a short timeout.
2. If the response contains the `X-Burp-AI-Agent: mcp` header, the occupant is a previous Custom AI Agent instance (same Burp process restarted, duplicate extension load, etc.). The supervisor issues `POST /__mcp/shutdown` with the active bearer token and waits 1 s before retrying the bind. Up to 3 takeover attempts are made.
3. If the occupant does not advertise the marker header, the supervisor refuses to proceed and surfaces a `BindException` in the UI — no shutdown is sent to unknown processes.

Outside of port conflicts, the supervisor monitors the listener and attempts up to 4 automatic restarts with a 2-second delay on unexpected termination.

## Next Steps

* [Security Model](security-model.md)
* [Tools Reference](tools-reference.md)
* [Issue Creation (MCP)](issue-create.md)
