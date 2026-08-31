# MCP Security Model

The MCP server follows a default-deny posture because external clients can request actions inside Burp.

## 1. Local-Only Binding

By default, server bind address is `127.0.0.1`. Only local processes can connect unless you explicitly enable external access.

## 2. Token Authentication

* Token is auto-generated on first run with `SecureRandom` (32 bytes, base64) and can be rotated in **MCP Server** settings.
* When **External Access** is enabled, every non-health request must include `Authorization: Bearer <token>`; `GET /__mcp/health` remains unauthenticated for liveness checks.
* When external access is disabled (default), loopback clients are **not** required to send the bearer token — the server instead rejects the request when the `Host`/`Origin`/`Referer` look browser-originated or non-local (see §4). The `POST /__mcp/shutdown` administrative endpoint still requires a credential in both modes — either the bearer token or a takeover proof of possession (see §4).

## 3. Tool Gating (Safe vs Unsafe)

* **Safe tools**: read-only operations; many are catalog-enabled by default, subject to their per-tool toggle.
* **Unsafe tools**: state/traffic modifying operations. The global unsafe master switch is off by default, so they cannot run even if an individual descriptor (such as `http1_request`) is catalog-enabled.

{% hint style="warning" %}
Unsafe tools can modify Burp state and generate outbound traffic. Enable only when needed and only for trusted clients.
{% endhint %}

This is a **capability** switch — whether a tool may ever run at all. It is independent of the SEC-06
confirmation tier in §7, which decides whether the extension's own AI may run a tool *without asking
you*. Neither is derivable from the other: `ai_analyze` and `ai_passive_scan` ask on every call
without being unsafe tools at all.

### Build Variant Surface

The build you load is the first gate on what MCP can reach:

* The **BApp Store build** exposes **only the 8 extension-native AI tools** (`status`, `issue_create`, `ai_analyze`, `ai_passive_scan`, `ai_findings_recent`, `redact_preview`, `ai_audit_query`, `ai_backends_list`). It does **not** expose generic Burp/Montoya tools (proxy history, repeater, scanner, scope, site map, intruder, Collaborator, utilities, etc.) — for those, run PortSwigger's official Burp MCP Server.
* The **full build** (GitHub releases) exposes all 59 MCP tools.

The `ai_analyze` and `ai_passive_scan` handlers currently check Burp's `api.ai().isEnabled()` unconditionally before invoking the extension supervisor. As a result, those two MCP tools are unavailable on Community or with **Use AI for extensions** off even when an independent backend is selected. This is narrower than the normal chat/scanner backend gate, which only blocks the `burp-ai` backend.

### Scope Restriction

When **Restrict MCP tools to in-scope hosts** (`mcpScopeOnly`) is enabled in **MCP Server** settings, every scope-aware tool (proxy history, site map, the HTTP request senders, scanner, etc.) rejects targets outside Burp's defined scope before acting on them. This confines an external MCP client to the same scope you are testing, so it cannot reach out-of-scope hosts through Burp. The generic tools live in the full build; the BApp Store build only ships the 8 extension-native AI tools.

## 4. Origin and Host Validation

In local mode, the access-control gate validates `Host`, `Origin`, and `Referer`, and rejects browser-like `User-Agent` requests when no acceptable loopback origin is present. In external mode, Ktor CORS owns origin filtering and the bearer gate owns authorization; **Allowed Origins** configures that CORS list. Leaving it empty in external mode calls `anyHost()`, so set an explicit list when browser-origin restrictions matter. The unauthenticated health route is the documented exception.

### Administrative Endpoints

* `GET /__mcp/health` — returns `"ok"` without authentication. In local mode it also returns `X-Burp-AI-Agent: mcp`; external mode deliberately omits that identifying header. The MCP Supervisor uses the route as a bind-conflict liveness probe.
* `POST /__mcp/shutdown` — requires either `Authorization: Bearer <token>` or a valid `X-Mcp-Takeover-Proof` header. Used by a new MCP server instance to ask a colliding older instance to release the port during a graceful takeover.

  **The automatic takeover never sends the token.** It presents an HMAC-SHA256 proof of possession keyed by the MCP token and bound to the target host, port and a 10-second window (`McpTakeoverProof`), so a local process that squats the port and echoes `X-Burp-AI-Agent: mcp` cannot harvest a credential that would grant it full MCP tool access to your Burp. The residual is narrow and accepted: within its window that proof could be replayed to shut down the freshly-bound server — a denial of service by a process that is already denying service by holding the port. Under TLS on a loopback bind, the client additionally pins the server certificate to the one in its own keystore and fails closed if no pin can be computed. Automatic takeover is not supported for a non-loopback TLS bind: free the port manually and restart MCP.

## 5. Privacy Mode Integration

MCP tool results pass through the active read-time privacy mode before leaving Burp. Structured producers also sanitize recognized headers, URLs, and cookie-typed parameters before serialization.

* `STRICT`: recognized token/cookie filtering plus host anonymization on covered carriers.
* `BALANCED`: recognized token/cookie filtering with real hostnames.
* `OFF`: built-in filtering disabled; configured custom redaction patterns still run.

This is not a field-independent DLP guarantee. In particular, raw HTTP/serialized URL carriers, generic `value` fields, and stored scanner issue details have [documented boundaries](../privacy/limitations.md#redaction-coverage-and-known-boundaries). Use `redact_preview` for representative payload shapes and keep bulk traffic tools in a confirmation tier.

## 6. TLS

TLS is optional for loopback-only operation and mandatory when **External Access** is enabled. It can use:

* Auto-generated self-signed certificate (default), or
* Custom PKCS12 keystores for enterprise environments.

When auto-generation is active, the extension shells out to the JDK-bundled `keytool` (no BouncyCastle dependency) and produces a PKCS12 keystore with:

* 2048-bit RSA key
* `SHA256withRSA` signature
* 365-day validity
* Subject `CN=burp-mcp`
* Stored at `~/.burp-ai-agent/certs/mcp-keystore.p12`

Certificate generation depends on the `keytool` binary from the active Java runtime. The project builds against Java 21; use that supported toolchain (or the compatible runtime bundled with Burp) rather than assuming a blanket Java 8–25 runtime contract.

## 7. Tool-Call Confirmation (SEC-06)

§1-§6 govern what an **external MCP client** can reach. This section governs something different:
what the extension's **own AI** may do when it emits a tool call in its response text. Those calls
originate with the model, not with you, so they carry their own trust boundary.

A tool call parsed out of model output does not execute against Burp until you decide. Every tool
carries a required security tier:

| Tier | Behaviour |
| :--- | :--- |
| `AUTO` | Runs with no user decision. Requires read-only **and** bounded output. |
| `CONFIRM` | Asks, and offers **Approve for session** — scoped to the current chat. |
| `CONFIRM_EACH` | Asks on every single call. No session memory in either direction. |

**Resolution fails closed.** A tool name the catalog does not recognise resolves to `CONFIRM_EACH`,
never to `AUTO`. Every `ext:`-namespaced external tool resolves to `CONFIRM_EACH` before the catalog
is consulted at all, so an external tool can never inherit a built-in tool's silent tier.

**Read-only is not sufficient for `AUTO`.** `proxy_http_history`, `site_map` and `scanner_issues` are
read-only yet ask, because what they return is bulk attacker-controlled traffic entering model
context.

**The prompt is a card in the chat transcript, not a modal dialog.** It names the tool, shows the
arguments the model supplied, and offers **Approve once**, **Approve for session**, **Deny** and
**Deny for session**. Arguments are truncated for display only — the full arguments are sent if you
approve.

**Session approvals are narrow and impermanent.** They are scoped to one chat session, discarded by
**Clear Chat** and by starting a new session, and held in memory only — restarting Burp, reloading
the extension or switching Burp project all clear them.

**Denying is not an error.** The model receives a neutral "this tool call was not authorised, do not
retry it, continue with the information you already have" result, so it does not treat the refusal as
a malfunction to work around.

**Every decision is recorded** — including automatic runs and denials — as an audit event plus a line
in Burp's **Output** tab. Audit logging is off by default, so the Output line is the record most
users see.

### Credential Storage

Both the MCP bearer token and the TLS keystore password are persisted in Burp's preferences store (keys `mcp.token` and `mcp.tls.keystore.password`). They are encrypted with the extension's `ENC1:` format, but the AES master key is stored in the same Burp preferences domain. Treat access to the preferences store or its exports as credential access.

* If Burp preferences or an export could leak (shared backups, multi-user hosts), treat both the bearer token and the TLS keystore as compromised and rotate them.
* The MCP bearer token is generated with `SecureRandom` (32 bytes, base64). Rotate it whenever an external client is decommissioned.
* For high-assurance setups, generate the keystore offline with your own `keytool` invocation and point the extension at it via settings, so the password never touches Burp preferences.

## 8. Tool Execution Audit

Every MCP tool call is logged to the [AI Request Logger](../privacy/ai-request-logger.md) with:

* **Policy decision**: `allowed`, `disabled`, `unsafe_blocked`, `pro_only`, `concurrency_limited`.
* **Argument hash** (`argsSha256`): SHA-256 checksum of the tool arguments for correlation and comparison.
* **Result hash** (`resultSha256`): SHA-256 checksum of the tool output.
* **Status**: `ok`, `error`, `blocked`.
* **Duration**: Execution time in milliseconds.

This provides execution telemetry for tool invocations from chat tool chaining and external MCP clients. The hashes are unkeyed and stored beside the record, so they do not authenticate it against a writer who can edit both value and hash. The AI Request Logger is enabled by default but bounded in memory; durable retention requires its rolling JSONL option or Audit Logging, and filesystem protection remains the operator's responsibility.

## Related Pages

* [MCP Overview](overview.md)
* [MCP Tools Reference](tools-reference.md)
* [Privacy Modes](../privacy/privacy-modes.md)
