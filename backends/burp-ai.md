# Burp AI (Built-in)

The **Burp AI** backend is an in-process backend that runs through Burp Suite Professional's built-in AI capability. It requires no external URL, API key, or CLI and produces the lowest-latency response path among all backends because no HTTP or child process boundary is crossed.

{% hint style="info" %}
This backend is only available on Burp Suite **Professional** with AI credits and the **Use AI for extensions** option enabled. It is not available on Burp Suite Community.
{% endhint %}

## Prerequisites

1. Burp Suite **Professional** with an active AI subscription (AI credits).
2. In Burp: **Settings → Burp AI → Use AI for extensions** must be set to **ON**.

When *Use AI for extensions* is off, the extension's supervisor refuses to start a Burp AI session and you'll see `AI: Offline` in the top bar even when the backend is selected. Switching it on takes effect immediately without restarting Burp.

## Selecting Burp AI

1. Open **Custom AI Agent → Settings → AI Backend**.
2. In **Preferred Backend** choose **Burp AI (built-in)**.
3. Optionally click **Test connection**. A healthy backend reports `Healthy`; if Burp AI is disabled in Burp settings the health check returns `Unavailable: Burp AI is not enabled. Enable 'Use AI' in Burp Suite settings.`

There is no URL, model, token, or custom command — configuration lives entirely inside Burp's own AI settings.

## Capabilities

| Capability | Value |
| :--- | :--- |
| Streaming | No — Burp returns a full response from `api.ai().prompt().execute(...)` in one call. |
| JSON mode | Enforced via the prompt: when `jsonMode=true` the extension appends *"IMPORTANT: Respond ONLY with valid JSON."* to the user message. |
| System role | Yes. Agent profiles are delivered as `Message.systemMessage(...)` and precede the conversation history. |
| Auto-start | Not applicable (no process to launch). |
| Temperature | `0.0` when **Determinism Mode** is on, otherwise `0.3`. |

## Privacy Posture

Burp AI keeps requests inside Burp's own AI route. The plugin does not open additional outbound connections when this backend is selected — the prompt, context, and conversation history all flow through the Montoya `api.ai()` channel. Privacy-mode redaction is still applied to the payload before handoff, so cookies, tokens, and (in STRICT mode) hostnames are stripped just as with any other backend.

See [Privacy Modes](../privacy/privacy-modes.md) for what gets redacted.

## Limitations

* Not available on Burp Community.
* Streaming UI indicators in the chat panel appear as a single chunk because the backend returns the full response at once.
* There is no per-backend timeout setting; request timing is governed by Burp Pro's own AI limits.
* The extension's HTTP retry/circuit breaker does not wrap this backend — retries fall back to whatever Burp Pro does internally.
* Tool-chain execution still runs through the MCP catalog like any other backend, but the response text is produced in a single call rather than streamed.

## Troubleshooting

* **`AI: Offline` in the top bar with Burp Pro running** — open **Burp Settings → Burp AI** and toggle **Use AI for extensions** on. The plugin polls `api.ai().isEnabled()` on every health cycle and will pick the change up automatically.
* **"Unsupported backend" error from chat** — this was a fixed regression in v0.5.0; make sure you are on v0.5.0 or newer.
* **AI credits exhausted** — Burp Pro surfaces quota errors directly; the plugin relays them as an `ERROR` entry in the [AI Request Logger](../privacy/ai-request-logger.md).
* **`api.ai().isEnabled()` throws on custom builds** — on older Montoya API versions the `ai()` surface may be missing; the backend falls back to `Unavailable` and disappears from the Preferred Backend dropdown automatically.

## Related Pages

* [First Run Checklist](../getting-started/first-run-checklist.md)
* [Backends Overview](overview.md)
* [Burp Integration](../user-guide/burp-integration.md)
* [Troubleshooting](../reference/troubleshooting.md)
