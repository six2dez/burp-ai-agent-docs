# Anthropic (API)

The Anthropic backend calls the native [Anthropic Messages API](https://docs.anthropic.com/en/api/messages) (`/v1/messages`) directly — Claude models without a CLI wrapper. Unlike the [Claude CLI](claude-cli.md) backend (which shells out to the `claude` binary), this is a first-class HTTP backend: requests go through Burp's own Montoya HTTP stack, so all Anthropic traffic is visible in **Proxy > HTTP history**. Introduced in v0.9.0.

## Requirements

* An Anthropic API key (`sk-ant-…`) from [console.anthropic.com](https://console.anthropic.com).

## Setup

1. Open the **AI Backend** settings tab and select **Anthropic** as the Preferred Backend.
2. Enter your **API key**. It is encrypted at rest (AES-256-GCM, `ENC1:`-prefixed) and never written to logs or exported settings.
3. Set the **Model** — a free-form field, so you can use any current Anthropic model without an extension update. Defaults to a current Claude Sonnet alias (e.g. `claude-3-5-sonnet-20241022`).
4. Click **Save**, then **Test connection** to confirm the key and model are accepted.

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `Anthropic` |
| **Anthropic API Key** | `sk-ant-…` (stored AES-256-GCM encrypted) |
| **Anthropic Model** | free-form; default `claude-3-5-sonnet-20241022` |
| **Base URL** | `https://api.anthropic.com` (`/v1/messages`) |
| **Timeout** | 30 s (raise for large prompts or slow links) |

## Notes

* **Proxy-visible by design.** All requests to `api.anthropic.com` route through `MontoyaHttpTransport` — not a vendored Anthropic SDK — so they respect Burp's upstream proxy, TLS, and logging and appear in Proxy > HTTP history like any other request (#69).
* **Token counting.** Anthropic's usage fields (input / output / cache-read / cache-write) are surfaced per request and feed the [token-budget guardrails](../user-guide/token-management.md).
* **Encrypted key.** The API key is encrypted with a per-install master key; the plaintext value never appears in logs or exported settings.
* **Scope (v0.9.0).** Ships streaming (single-chunk, proxy-visible — the transport buffers the response, matching every other HTTP backend), token counting, model selection, and the encrypted key. **Native tool-use and prompt caching are deferred to a future release.**

## Error Handling

A `400` response whose body mentions `model` surfaces a specific message — *"Anthropic rejected the model ID — check Settings > Anthropic > Model"* — instead of a generic error, so a model-name typo is obvious.

## Retry Behavior

Like the other HTTP backends, Anthropic requests retry on transient network errors with bounded stepped backoff and are wrapped in the shared circuit breaker (5 consecutive failures open it for 30 s before a half-open probe). See [Backends Overview → Retry Behavior](overview.md#retry-behavior).

## Related Pages

* [Backends Overview](overview.md)
* [Claude CLI](claude-cli.md) — the CLI-based alternative
* [Token Usage & Cost Management](../user-guide/token-management.md)
* [Backend Troubleshooting](troubleshooting.md)
