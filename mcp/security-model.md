# MCP Security Model

The MCP server follows a default-deny posture because external clients can request actions inside Burp.

## 1. Local-Only Binding

By default, server bind address is `127.0.0.1`. Only local processes can connect unless you explicitly enable external access.

## 2. Token Authentication

* Token is auto-generated on first run with `SecureRandom` (32 bytes, base64) and can be rotated in **MCP Server** settings.
* When **External Access** is enabled, every request must include `Authorization: Bearer <token>`.
* When external access is disabled (default), loopback clients are **not** required to send the bearer token — the server instead rejects the request when the `Host`/`Origin`/`Referer` look browser-originated or non-local (see §4). The bearer token is still mandatory for the `POST /__mcp/shutdown` administrative endpoint in both modes.

## 3. Tool Gating (Safe vs Unsafe)

* **Safe tools**: read-only operations, enabled by default.
* **Unsafe tools**: state/traffic modifying operations, disabled by default.

{% hint style="warning" %}
Unsafe tools can modify Burp state and generate outbound traffic. Enable only when needed and only for trusted clients.
{% endhint %}

## 4. Origin and Host Validation

The server validates `Host`, `Origin`, `Referer` and rejects browser `User-Agent` strings (configurable via **Allowed Origins**) to reduce CSRF and cross-origin abuse paths, even for loopback clients.

### Administrative Endpoints

* `GET /__mcp/health` — returns `"ok"` and the `X-Burp-AI-Agent: mcp` header. Used internally by the MCP Supervisor to detect another live Custom AI Agent MCP instance on the same port (takeover probe).
* `POST /__mcp/shutdown` — requires `Authorization: Bearer <token>`. Used by a new MCP server instance to ask a colliding older instance to release the port during a graceful takeover.

## 5. Privacy Mode Integration

MCP output is filtered through active privacy mode before leaving Burp.

* `STRICT`: host anonymization + sensitive token/cookie filtering.
* `BALANCED`: token/cookie filtering with real hostnames.
* `OFF`: no redaction.

## 6. TLS (Optional)

TLS can be enabled for external access:

* Auto-generated self-signed certificate (default), or
* Custom PKCS12 keystores for enterprise environments.

When auto-generation is active, the extension shells out to the JDK-bundled `keytool` (no BouncyCastle dependency) and produces a PKCS12 keystore with:

* 2048-bit RSA key
* `SHA256withRSA` signature
* 365-day validity
* Subject `CN=burp-mcp`
* Stored at `~/.burp-ai-agent/certs/mcp-keystore.p12`

This implementation works on JDK 8 through JDK 25+ without additional dependencies.

### Credential Storage

Both the MCP bearer token and the TLS keystore password are persisted in Burp's preferences store (keys `mcp.token` and `mcp.tls.keystore.password`). Burp preferences are stored in the user's project file and are only as protected as that file.

* If the project file or a preferences export could leak (shared backups, multi-user hosts), treat both the bearer token and the TLS keystore as compromised and rotate them.
* The MCP bearer token is generated with `SecureRandom` (32 bytes, base64). Rotate it whenever an external client is decommissioned.
* For high-assurance setups, generate the keystore offline with your own `keytool` invocation and point the extension at it via settings, so the password never touches Burp preferences.

## 7. Tool Execution Audit

Every MCP tool call is logged to the [AI Request Logger](../privacy/ai-request-logger.md) with:

* **Policy decision**: `allowed`, `disabled`, `unsafe_blocked`, `pro_only`, `concurrency_limited`.
* **Argument hash** (`argsSha256`): SHA-256 of the tool arguments for tamper detection.
* **Result hash** (`resultSha256`): SHA-256 of the tool output.
* **Status**: `ok`, `error`, `blocked`.
* **Duration**: Execution time in milliseconds.

This provides a complete audit trail for all tool invocations regardless of whether they originate from chat tool chaining or external MCP clients.

## Related Pages

* [MCP Overview](overview.md)
* [MCP Tools Reference](tools-reference.md)
* [Privacy Modes](../privacy/privacy-modes.md)
