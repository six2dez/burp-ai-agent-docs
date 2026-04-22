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
| **Preferred Backend** | `NVIDIA NIM` | Select backend. |
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

Transient network failures trigger automatic exponential-backoff retries (max 6 attempts). Each retry is recorded in the [AI Request Logger](../privacy/ai-request-logger.md) as a `RETRY` activity.

## Related Pages

* [Backends Overview](overview.md)
* [Generic (OpenAI-compatible)](openai-compatible.md)
* [Troubleshooting](../reference/troubleshooting.md)
