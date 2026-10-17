# NVIDIA NIM

NVIDIA NIM (`integrate.api.nvidia.com`) hosts a range of open and proprietary models behind an OpenAI-compatible chat-completions interface. The extension targets `/v1/chat/completions` with the configured bearer token.

## Requirements

* An NVIDIA Developer account and an API key (starts with `nvapi-…`).
* Network access to `integrate.api.nvidia.com`.

## Setup

1. Sign up at [build.nvidia.com](https://build.nvidia.com/) and generate an API key.
2. Pick a model (for example `moonshotai/kimi-k2.5`).
3. Configure the backend in the **AI Backend** settings tab.

## Configuration

| Setting | Default | Description |
| :--- | :--- | :--- |
| **Preferred Backend** | `nvidia-nim` | Select backend. |
| **Base URL** | `https://integrate.api.nvidia.com` | NVIDIA-hosted endpoint; override only when targeting a self-hosted NIM. |
| **Model** | *(empty)* | Model identifier, e.g. `moonshotai/kimi-k2.5`. |
| **API Key** | *(empty)* | Your `nvapi-…` token. Sent as `Authorization: Bearer …`. |
| **Extra Headers** | *(empty)* | Optional extra `Header: value` lines if a gateway requires them. |
| **Timeout** | `120` | Request timeout in seconds. |

A working baseline:

```text
Backend: NVIDIA NIM
Base URL: https://integrate.api.nvidia.com
Model: moonshotai/kimi-k2.5
API Key: nvapi-...
```

## Health Check Traffic

The NVIDIA NIM health check is a free `GET /v1/models` against the configured Base URL, with the API key as a bearer token, sent through Burp's networking (the same Montoya transport as chat and scanner requests), so Burp's upstream proxy settings apply to it. It contains no captured Burp context and is visible in Proxy history. It runs no billed completion. It runs on the same schedule as every remote provider: in the BApp Store build only on demand (Save, a backend change, a pill click, or a chat reply); in the GitHub full build at startup, after a settings or backend change and on a pill click, never on a timer. Like every AI request, it is refused while Burp's "Use AI" gate refuses the backend (see [Which Build?](../getting-started/which-build.md)).

## Response Delivery Limitation

Normal NIM requests currently include `stream: true`, but the production Montoya transport buffers the response and the shared parser expects one JSON document before calling the UI once. It does not decode SSE `data:` frames. If the selected NIM endpoint honors streaming strictly, the response may fail JSON parsing; the current implementation should not be described as functional token-by-token streaming.

## Privacy Considerations

NVIDIA NIM is a cloud backend. The same privacy guidance as other cloud providers applies:

* Keep privacy mode at `STRICT` or `BALANCED` (the default) for real targets.
* Review the context preview dialog before sending auto-captured traffic.
* Review the [Privacy Modes](../privacy/privacy-modes.md) page for redaction patterns.

## Output Token Limits

The extension sets `max_tokens` automatically per request type:

| Request Type | `max_tokens` |
| :--- | :--- |
| **Chat** | 4096 |
| **Scanner (single request)** | 2048 |
| **Scanner (batch analysis)** | 4096 |
| **Payload generation** | 1024 |

## Troubleshooting

{% hint style="tip" %}
* `401 Unauthorized`: verify the API key is a valid `nvapi-…` token and not expired.
* `404 Not Found` on the model: confirm the model ID exactly matches NVIDIA's catalog.
* Slow first token: NIM models are shared; cold starts are expected.
* Extra headers: only add them if your organization routes requests through a gateway.
{% endhint %}

## Retry Behavior

Classified transient transport exceptions trigger up to 6 total attempts with five possible delays (`500 / 1000 / 1500 / 2000 / 3000 ms`). HTTP error responses are not retried inside the same call, although 429/5xx responses count toward the connection's circuit breaker. Each actual retry is recorded in the [AI Request Logger](../privacy/ai-request-logger.md) as a `RETRY` activity.

## Related Pages

* [Backends Overview](overview.md)
* [Generic (OpenAI-compatible)](openai-compatible.md)
* [Troubleshooting](../reference/troubleshooting.md)
