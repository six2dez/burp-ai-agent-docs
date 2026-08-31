# Perplexity

Perplexity (`https://api.perplexity.ai`) hosts the **Sonar** family of web-aware reasoning models behind an OpenAI-style chat-completions interface. The extension ships a dedicated factory for it because Perplexity diverges from the standard OpenAI shape in two material ways:

* The chat-completions endpoint is `POST /chat/completions` — **without** the `/v1/` prefix used by NVIDIA NIM, OpenAI, and most compatible providers.
* The Sonar API does **not** accept `response_format: json_object`, so JSON mode is disabled at the protocol level. Prompts that require structured output still work through prompt-level instruction.

{% hint style="warning" %}
Do not point the [Generic (OpenAI-compatible)](openai-compatible.md) backend at `https://api.perplexity.ai` — Generic targets `/v1/chat/completions` and will return `404 Not Found`. Use this dedicated Perplexity backend instead.
{% endhint %}

## Requirements

* A Perplexity API account and an API key (starts with `pplx-…`).
* Network access to `api.perplexity.ai`.

## Setup

1. Sign in at [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api) and generate an API key.
2. Pick a Sonar model (for example `sonar-pro`).
3. Configure the backend in the **AI Backend** settings tab.

<figure><img src="../.gitbook/assets/Screenshot 2026-05-15 at 10.41.59.png" alt=""><figcaption></figcaption></figure>

## Configuration

| Setting               | Default                     | Description                                                      |
| --------------------- | --------------------------- | ---------------------------------------------------------------- |
| **Preferred Backend** | `perplexity`                | Select backend.                                                  |
| **Base URL**          | `https://api.perplexity.ai` | Override only if you proxy Perplexity through your own gateway.  |
| **Model**             | _(empty)_                   | Sonar model identifier, e.g. `sonar-pro`.                        |
| **API Key**           | _(empty)_                   | Your `pplx-…` token. Sent as `Authorization: Bearer …`.          |
| **Extra Headers**     | _(empty)_                   | Optional extra `Header: value` lines if a gateway requires them. |
| **Timeout**           | `120`                       | Request timeout in seconds (accepted range: 30-3600).            |

A working baseline:

```
Backend: Perplexity
Base URL: https://api.perplexity.ai
Model: sonar-pro
API Key: pplx-...
```

## Supported Models

The extension does not maintain a model allowlist: it sends the free-form **Model** value unchanged. Use a currently supported Sonar model ID from Perplexity's API catalog; `sonar-pro` is only an example, not an extension default.

## Capabilities

| Capability  | Value                                                                                                                  |
| ----------- | ---------------------------------------------------------------------------------------------------------------------- |
| Streaming   | Not functionally decoded. The request currently sets `stream: true`, but the production parser expects one complete JSON document and emits one UI chunk. |
| JSON mode   | **No** — `response_format=json_object` is not supported by Sonar. Use prompt-level instructions for structured output. |
| System role | Yes — agent profiles are delivered as the `system` message.                                                            |
| Auto-start  | Not applicable (cloud backend).                                                                                        |

The lack of protocol-level JSON mode means structured scanner workflows—notably batch passive analysis and adaptive payload generation—fall back to a text-mode parser. The parser scans the model output for fenced JSON blocks first, then a top-level `{` / `[`, then individual field regexes as a last resort. It recovers from prose preambles and markdown wrappers but is more brittle than a `response_format=json_object` constraint: malformed JSON or schemas with unexpected fields are silently dropped. Keep confidence thresholds conservative when using Perplexity for scanner workflows.

There is also a response-framing limitation: the normal request sets `stream: true`, while the Montoya transport buffers the body and the shared backend parser calls `readTree` on that body as one JSON document. Actual SSE `data:` frames are not decoded on this path. A provider/gateway that returns a non-streaming JSON envelope can work; strict SSE output can fail parsing.

## Privacy Considerations

Perplexity is a cloud backend. The same guidance as other cloud providers applies:

* Keep privacy mode at `STRICT` or `BALANCED` (the default) for real targets.
* Review the context preview dialog before sending auto-captured traffic.
* Review the [Privacy Modes](../privacy/privacy-modes.md) page for redaction patterns.

## Health Check Traffic

While Perplexity is the selected backend, the top-bar status check runs about every 5 seconds and sends a fixed, non-streaming `"Hey"` completion with `max_tokens: 16`. The current health-check provider uses the shared direct HTTP client rather than Burp's Montoya transport. It contains no captured Burp context, but it bypasses Burp's upstream proxy/Proxy history and may count toward quota or billing. Normal chat and scanner requests still use Montoya.

## Output Token Limits

The extension sets `max_tokens` automatically per request type:

| Request Type                 | `max_tokens` |
| ---------------------------- | ------------ |
| **Chat**                     | 4096         |
| **Scanner (single request)** | 2048         |
| **Scanner (batch analysis)** | 4096         |
| **Payload generation**       | 1024         |

## Troubleshooting

{% hint style="info" %}
* `401 Unauthorized`: verify the API key is a valid `pplx-…` token and not expired.
* `404 Not Found`: confirm the Base URL is `https://api.perplexity.ai` (no `/v1` suffix). The factory targets `/chat/completions` directly.
* `400 Bad Request` mentioning `response_format`: a request tried to force JSON mode against Sonar. Disable the JSON-mode toggle for that request or switch to a backend that supports it.
* `model_not_found` / `invalid_model`: confirm the model ID matches Perplexity's catalog exactly.
* Slow first token: Sonar models are shared infrastructure; brief cold starts are expected.
* Extra headers: add them only if your organization routes requests through a gateway.
{% endhint %}

## Retry Behavior

Classified transient transport exceptions trigger up to 6 total attempts with five possible delays (`500 / 1000 / 1500 / 2000 / 3000 ms`). HTTP error responses are not retried inside the same call, although 429/5xx responses count toward the connection's circuit breaker. Each actual retry is recorded in the [AI Request Logger](../privacy/ai-request-logger.md) as a `RETRY` activity. After 5 recorded transient failures, that connection allows one half-open model request after 30 seconds.

## Related Pages

* [Backends Overview](overview.md)
* [Generic (OpenAI-compatible)](openai-compatible.md)
* [Troubleshooting](../reference/troubleshooting.md)
