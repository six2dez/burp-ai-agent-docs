# External MCP Servers

Beyond Burp's built-in MCP tools, Custom AI Agent can connect to **external or custom MCP servers** over **SSE** (HTTP) or **stdio** (local process) transports. Their tools appear alongside the built-in Burp tools in the agent's tool preamble, namespaced `ext:<server>:<tool>`. Introduced in v0.9.0 (closes #41).

## Setup

1. Open **Settings > MCP > External Servers**.
2. Click **Add** and choose a transport:
   * **SSE** — enter the server's SSE URL (e.g. `http://127.0.0.1:3000/sse`) and, if required, a bearer token.
   * **stdio** — enter the executable command (e.g. `/usr/local/bin/my-mcp-server`). stdio is **off by default** (it launches a local process) — enable it explicitly and confirm the local-process warning first.
3. Click **Connect**. The Status column shows `Connected (N tools)`, and the server's tools become callable as `ext:<server>:<tool>` and listed in the agent's tool preamble.

## Transport Types

| Transport | Use when | Example |
| :--- | :--- | :--- |
| **SSE** | Remote or local HTTP-based MCP server | `http://127.0.0.1:3000/sse` |
| **stdio** | Local process launched by the extension | `/usr/local/bin/my-mcp-server` |

## Security Model

{% hint style="warning" %}
External MCP servers are **untrusted by default**. Their output is wrapped in an explicit trust-boundary marker before it ever enters the AI prompt, so a compromised or malicious server cannot smuggle instructions into the agent.
{% endhint %}

* **Encrypted auth tokens.** SSE bearer tokens are stored encrypted at rest (AES-256-GCM, `ENC1:`-prefixed) — the same path as every other API key — masked in the UI behind a show/hide toggle, and never logged.
* **Trust-boundary wrapping.** Every external tool result is wrapped as `[EXTERNAL-TOOL-RESULT:<server>]…[/EXTERNAL-TOOL-RESULT]` (with close-marker escaping), marking it as untrusted data rather than agent instructions — a prompt-injection guard.
* **SSRF guard.** Configuring an external URL that resolves to a non-loopback private/link-local address triggers the same soft SSRF warning used for backend URLs — non-blocking, so deliberate internal use is still possible.
* **Audit logging.** Every external tool invocation is recorded in the [audit log](../privacy/audit-logging.md) (when enabled) with the server name, tool name, and a result summary.

## Related Pages

* [MCP Overview](overview.md)
* [Security Model](security-model.md)
* [Tools Reference](tools-reference.md)
* [Audit Logging](../privacy/audit-logging.md)
