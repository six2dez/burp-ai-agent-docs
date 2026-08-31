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

* **BApp Store build** (`./gradlew shadowJar -PstoreBuild=true` → `Custom-AI-Agent-<version>.jar`): registers **only the 8 extension-native AI tools** — `status`, `issue_create`, `ai_analyze`, `ai_passive_scan`, `ai_findings_recent`, `redact_preview`, `ai_audit_query`, `ai_backends_list`. Generic Burp/Montoya tools (proxy history, repeater, scanner, scope, site map, intruder, Collaborator, utilities, etc.) are intentionally **not** exposed here. For those, use PortSwigger's official Burp MCP Server alongside this extension.
* **Full build** (`./gradlew shadowJar` → `Custom-AI-Agent-full-<version>.jar`, GitHub releases): registers **all 59 MCP tools**, including the generic Montoya tools above.

A compile-time `BuildFlags.STORE_BUILD` constant gates which tools register. Independent backends continue to work for chat and scanner pipelines when Burp's built-in AI is off. The current `ai_analyze` and `ai_passive_scan` MCP handlers are an exception: both call Burp's `api.ai().isEnabled()` unconditionally, so those two tools refuse on Community or with **Use AI for extensions** disabled regardless of the selected backend.

When loaded, the extension appears in Burp's **Extensions** list and as a Suite tab titled **Custom AI Agent** (named that way to distinguish it from Burp's built-in "Burp AI" provider).

## Features

* SSE and optional STDIO transport.
* 59 MCP tools across Burp workflows in the full build (8 extension-native AI tools in the BApp Store build); see [Tools Reference](tools-reference.md).
* Unsafe-tool gating with per-tool toggles.
* Optional **Restrict MCP tools to in-scope hosts** (`mcpScopeOnly`) that confines every scope-aware tool to Burp's defined scope.
* Configurable request limiter and body-size caps.
* Proxy-history preprocessing pipeline (binary filter, size cap, content-type allowlist, newest-first, raw opt-in). See [MCP Proxy History Preprocessing](../reference/settings-reference.md#mcp-proxy-history-preprocessing).
* Administrative endpoints (`GET /__mcp/health`, `POST /__mcp/shutdown`) used for health checks and bounded takeover.
* Auto-restart of the MCP listener on unexpected termination.
* Privacy-aware tool output filtering.
* Inline advisory banner in the **MCP Server** settings tab that surfaces risky combinations (external access without allowed origins, external access with **Enable Unsafe Tools** on, etc.). See [UI Tour → Advisory Banner (SubtleNotice)](../user-guide/ui-tour.md#advisory-banner-subtlenotice).

<figure><img src="../.gitbook/assets/Screenshot 2026-05-15 at 10.33.38.png" alt=""><figcaption></figcaption></figure>

## Administrative Endpoints

| Endpoint          | Method | Auth                            | Purpose                                                                                                                                                                                   |
| ----------------- | ------ | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/__mcp/health`   | GET    | None                            | Returns `"ok"`. Local mode also emits `X-Burp-AI-Agent: mcp`; external mode omits the identifying header. Used as a bind-conflict liveness probe. |
| `/__mcp/shutdown` | POST   | `Authorization: Bearer <token>` **or** `X-Mcp-Takeover-Proof` | Used during port takeover: a new MCP server instance sends this to ask a colliding older instance to release the port. Accepts either credential form and rejects a request carrying neither. The automatic takeover client sends only the proof (see §Port Takeover); the bearer form remains for manual use. |

## Port Conflict Handling and Takeover

When the MCP Supervisor starts a server and the port is already in use:

1. It probes `GET <scheme>://<host>:<port>/__mcp/health` with a short timeout.
2. In local mode, takeover proceeds only when the response contains `X-Burp-AI-Agent: mcp`. In external mode that header is deliberately absent, so a successful health response is treated as liveness without an identity claim. The supervisor then issues `POST /__mcp/shutdown`, waits 1 s, and retries the bind. Up to 3 takeover attempts are made.

   **The MCP token is never sent on this automatic request.** Bind-conflict takeover presents a **proof of possession** (`X-Mcp-Takeover-Proof`: HMAC-SHA256 keyed by the MCP token and bound to host, port and a 10-second window) instead of the token itself. The bearer form still works for an operator driving the endpoint by hand. A port squatter can replay the proof during the current/previous acceptance window to shut down a freshly bound server, but cannot turn it into the reusable bearer token; this is a bounded denial-of-service residual, not listener authentication.
3. In **local mode**, if the occupant does not advertise the marker header, the supervisor refuses to proceed and surfaces the bind failure — no shutdown is sent. In **external mode**, the deliberately headerless health response is only a liveness signal; the proof prevents bearer-token disclosure but does not establish listener identity.

For TLS takeover, certificate pinning is installed only for `localhost`, `127.0.0.1`, or `::1`. A non-loopback TLS bind is left running and the extension logs that the operator must free the port manually before restarting MCP.

Outside of port conflicts, the supervisor monitors the listener and attempts up to 4 automatic restarts with a 2-second delay on unexpected termination.

## Next Steps

* [Security Model](security-model.md)
* [Tools Reference](tools-reference.md)
* [Issue Creation (MCP)](issue-create.md)
