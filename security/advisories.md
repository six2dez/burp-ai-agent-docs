# Security Advisories

This page mirrors the advisories in the project's [`SECURITY.md`](https://github.com/six2dez/burp-ai-agent/blob/main/SECURITY.md). If the two ever disagree, `SECURITY.md` in the repository is authoritative.

Both defects below were confirmed by **running** the shipped code during a review of `0.9.2` on 2026-08-05, not by reading it. Both affect every published `0.9.x` release, and both are fixed in **1.0.0**.

{% hint style="warning" %}
**No CVE and no GHSA identifier has been issued for either finding.** Do not look for one — none exists at the time of writing.

If you find a further issue, report it **privately** using the instructions in [`SECURITY.md`](https://github.com/six2dez/burp-ai-agent/blob/main/SECURITY.md) rather than opening a public issue.
{% endhint %}

## SEC-04 — MCP access-control checks did not run on resolved routes

|              |                             |
| :----------- | :-------------------------- |
| **Affected** | 0.9.0, 0.9.1, 0.9.2         |
| **Fixed in** | 1.0.0                       |
| **Severity** | Critical                    |

The access-control interceptor was registered *after* the `routing` block in Ktor's `Call` phase, and Ktor runs same-phase interceptors in registration order, so any request whose route resolved was served by its handler before the checks ran.

**What that exposed:**

* With external MCP access enabled, an unauthenticated `POST /message` and an unauthenticated SSE connect both reached the MCP handler instead of being rejected with `401`. The listener accepted unauthenticated tool calls.
* In local mode, the `Origin`, `Host` and `User-Agent` checks and the `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy` and `Content-Security-Policy` response headers did not apply to matched routes, leaving the browser-origin defences inert.

Reproduction actually observed: in external mode, with no `Authorization` header, `POST /message` returned `400 "sessionId query parameter is not provided"` — the MCP handler's own error, proving the handler ran — rather than `401`. Only unmatched paths returned `401`.

**Precondition, stated honestly:** the MCP server binds to `127.0.0.1` by default, so the unauthenticated-listener exposure required the explicit external-access opt-in. The local-mode gap required no opt-in; it applied to every install running the MCP server.

{% hint style="danger" %}
**User action:** if you enabled external MCP access on 0.9.0, 0.9.1 or 0.9.2, treat that listener as having accepted **unauthenticated** tool calls for the entire period it was reachable beyond loopback, and **rotate the MCP bearer token**. Review Burp's own logs and your [audit log](../privacy/audit-logging.md) for tool calls you did not initiate.
{% endhint %}

## PRIV-05 — session cookies reached AI backends unredacted in STRICT and BALANCED

|              |                             |
| :----------- | :-------------------------- |
| **Affected** | 0.9.0, 0.9.1, 0.9.2         |
| **Fixed in** | 1.0.0                       |
| **Severity** | High                        |

The passive scanner emitted a dedicated cookies section into the prompt as bare `name=value` pairs, dropping the `Cookie:` header prefix that the redaction rule keyed on. Sensitive-key matching was an exact match against a fixed list, so real-world cookie names were not recognised.

**What that exposed:** cookie values named `JSESSIONID`, `PHPSESSID`, `connect.sid`, `auth_token`, `csrftoken` and `remember_me` were included verbatim in the prompt sent to the configured AI backend in **both STRICT and BALANCED** privacy modes. Only a cookie literally named `session` was caught.

{% hint style="danger" %}
**User action:** if you ran passive AI scanning on 0.9.0, 0.9.1 or 0.9.2 with a **non-local backend** (any hosted provider — Anthropic, OpenAI, Google, NVIDIA, Perplexity, GitHub, or any OpenAI-compatible endpoint you configured), treat every session cookie that passed through that scanning as **disclosed to that provider** and rotate it.

Cookies belonging to your clients' or employer's applications are included; rotation is their decision to make, so **tell them**.

If your backend was a local runner (Ollama, LM Studio) the data did not leave your machine.
{% endhint %}

## Supported versions

| Version | Supported                                  |
| :------ | :----------------------------------------- |
| 1.0.x   | Yes                                        |
| 0.9.x   | No — upgrade to 1.0.0                      |
| < 0.9   | No                                         |

## Related hardening in 1.0.0

Beyond the two advisories above, the 1.0.0 line closed a set of defects found by the same review. None of them was confirmed exploitable in a shipped release, so none carries an advisory — but they are the reason several behaviours described elsewhere in this documentation changed:

* **The MCP token is no longer sent during a port takeover.** A local process that squatted the MCP port and echoed the identity header could previously collect the bearer token. The takeover client now presents a proof of possession instead. See [Security Model §5](../mcp/security-model.md).
* **Model-emitted tool calls now require your approval.** See [Tool-Call Confirmation](../mcp/security-model.md#7-tool-call-confirmation-sec-06).
* **`SsrfGuard` classifies alternate IP notations.** A backend URL written as `http://2852039166/` (decimal for `169.254.169.254`) previously bypassed the private/link-local advisory warning by notation alone.
* **Shell arguments are quoted by allowlist.** A settings value containing shell metacharacters without whitespace (`foo;id`, `$(cmd)`) could previously reach `sh -c` unquoted.
* **The at-rest encryption caveat is now documented** rather than overstated. See [Secrets at Rest](../privacy/limitations.md#secrets-at-rest--what-the-encryption-does-not-do).
