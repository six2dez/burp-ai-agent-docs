# MCP Security Model

The MCP server follows a default-deny posture because external clients can request actions inside Burp.

## 1. Local-Only Binding

By default, server bind address is `127.0.0.1`. Only local processes can connect unless you explicitly enable external access.

## 2. Token Authentication

External access requires `Authorization: Bearer <token>`.

* Token is auto-generated on first run.
* Token can be rotated in **MCP Server** settings.
* Local loopback access does not require token by default.

## 3. Tool Gating (Safe vs Unsafe)

* **Safe tools**: read-only operations, enabled by default.
* **Unsafe tools**: state/traffic modifying operations, disabled by default.

{% hint style="warning" %}
Unsafe tools can modify Burp state and generate outbound traffic. Enable only when needed and only for trusted clients.
{% endhint %}

## 4. Origin and Host Validation

The server validates `Host` and checks suspicious `Origin` headers to reduce CSRF/cross-origin abuse paths.

## 5. Privacy Mode Integration

MCP output is filtered through active privacy mode before leaving Burp.

* `STRICT`: host anonymization + sensitive token/cookie filtering.
* `BALANCED`: token/cookie filtering with real hostnames.
* `OFF`: no redaction.

## 6. TLS (Optional)

TLS can be enabled for external access:

* auto-generated self-signed certificates, or
* custom PKCS12 keystores for enterprise environments.

### Credential Storage

Both the MCP bearer token and the TLS keystore password are persisted in Burp's preferences store (keys `mcp.bearer.token` and `mcp.tls.keystore.password`). Burp preferences are stored in the user's project file and are only as protected as that file.

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
