# Anthropic (API)

The Anthropic backend calls the native [Anthropic Messages API](https://docs.anthropic.com/en/api/messages) (`/v1/messages`) directly — Claude models without a CLI wrapper. Unlike the [Claude CLI](claude-cli.md) backend (which shells out to the `claude` binary), this is a first-class HTTP backend: requests go through Burp's own Montoya HTTP stack, so all Anthropic traffic is visible in **Proxy > HTTP history**. Introduced in v0.9.0.

## Requirements

* An Anthropic API key (`sk-ant-…`) from [console.anthropic.com](https://console.anthropic.com).

## Setup

1. Open the **AI Backend** settings tab and select `anthropic` as the Preferred Backend.
2. Enter your **API key**. It is encrypted at rest (AES-256-GCM, `ENC1:`-prefixed) and never written to logs or exported settings — see [Encrypted key](#notes) below for what that encryption does and does not protect against.
3. Set the **Model** — a free-form field, so you can use any current Anthropic model without an extension update. The current default is `claude-sonnet-4-6`.
4. Click **Save**, then **Test connection** to check that a key is configured. The current health path does not contact Anthropic or validate the model; use a real prompt for live validation.

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `anthropic` |
| **Anthropic API Key** | `sk-ant-…` (stored AES-256-GCM encrypted; master key is in Burp Preferences too — see Notes) |
| **Anthropic Model** | free-form; default `claude-sonnet-4-6` |
| **Base URL** | `https://api.anthropic.com` (`/v1/messages`) |
| **Timeout** | 120 s runtime default; not currently exposed as an Anthropic-specific UI field |

## Notes

* **Proxy-visible by design.** All requests to `api.anthropic.com` route through `MontoyaHttpTransport` — not a vendored Anthropic SDK — so they respect Burp's upstream proxy, TLS, and logging and appear in Proxy > HTTP history like any other request (#69).
* **Configuration-only health.** A non-empty API key maps to `Healthy` without a network request. `Test connection` and the top-bar `AI: OK` state therefore do not prove that Anthropic will accept the key or model; the first actual prompt is the live check.
* **Token counting.** Anthropic's usage fields (input / output / cache-read / cache-write) are surfaced per request and feed the [token-budget guardrails](../user-guide/token-management.md).
* **Encrypted key.** The API key is encrypted with a per-install master key and the plaintext value never appears in logs or exported settings. **The master key is itself stored in Burp Preferences, Base64-encoded, beside the ciphertext** (preference `secret.master.key.v1`), so this protects against casual inspection of a preferences file or an export — not against a local attacker who can read those preferences. See [Secrets at Rest](../privacy/limitations.md#secrets-at-rest--what-the-encryption-does-not-do).
* **Scope (v0.9.0).** Ships buffered, single-chunk, proxy-visible responses, token counting, model selection, and the encrypted key. **Native tool-use and prompt caching are deferred to a future release.**

## Error Handling

A `400` response whose body mentions `model` surfaces a specific message — *"Anthropic rejected the model ID — check Settings > Anthropic > Model"* — instead of a generic error, so a model-name typo is obvious.

## Retry Behavior

Anthropic retries classified transient transport exceptions for up to 6 total attempts (five delays: 500/1000/1500/2000/3000 ms). Each launched connection has a circuit breaker; 5 recorded transient failures open it for 30 seconds before one half-open model request. HTTP error responses are not retried in the same call, although 429/5xx responses count toward the breaker. See [Backends Overview → Retry Behavior](overview.md#retry-behavior).

## Related Pages

* [Backends Overview](overview.md)
* [Claude CLI](claude-cli.md) — the CLI-based alternative
* [Token Usage & Cost Management](../user-guide/token-management.md)
* [Backend Troubleshooting](troubleshooting.md)
