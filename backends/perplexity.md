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
| **Preferred Backend** | `Perplexity`                | Select backend.                                                  |
| **Base URL**          | `https://api.perplexity.ai` | Override only if you proxy Perplexity through your own gateway.  |
| **Model**             | _(empty)_                   | Sonar model identifier, e.g. `sonar-pro`.                        |
| **API Key**           | _(empty)_                   | Your `pplx-…` token. Sent as `Authorization: Bearer …`.          |
| **Extra Headers**     | _(empty)_                   | Optional extra `Header: value` lines if a gateway requires them. |
| **Timeout**           | `60`                        | Request timeout in seconds.                                      |

A working baseline:

```
Backend: Perplexity
Base URL: https://api.perplexity.ai
Model: sonar-pro
API Key: pplx-...
```

## Supported Models

The Sonar family (web-aware) plus reasoning variants:

* `sonar` — fast, lightweight.
* `sonar-pro` — higher-capability default.
* `sonar-reasoning` — chain-of-thought reasoning.
* `sonar-reasoning-pro` — extended reasoning.
* `sonar-deep-research` — multi-step research with broader retrieval.
* `r1-1776` — uncensored variant of DeepSeek R1.

Always cross-check the current model catalog on Perplexity's API page — names may change.

## Capabilities

| Capability  | Value                                                                                                                  |
| ----------- | ---------------------------------------------------------------------------------------------------------------------- |
| Streaming   | Yes (SSE).                                                                                                             |
| JSON mode   | **No** — `response_format=json_object` is not supported by Sonar. Use prompt-level instructions for structured output. |
| System role | Yes — agent profiles are delivered as the `system` message.                                                            |
| Auto-start  | Not applicable (cloud backend).                                                                                        |

The lack of JSON mode means features that rely on guaranteed JSON output — notably batch passive analysis and adaptive payload generation — fall back to a text-mode parser. The parser scans the model output for fenced JSON blocks first, then a top-level `{` / `[`, then individual field regexes as a last resort. It recovers from prose preambles and markdown wrappers but is more brittle than the strict `response_format=json_object` path: malformed JSON or schemas with unexpected fields are silently dropped. Keep confidence thresholds conservative when using Perplexity for scanner workflows.

## Privacy Considerations

Perplexity is a cloud backend. The same guidance as other cloud providers applies:

* Keep privacy mode at `STRICT` or `BALANCED` (the default) for real targets.
* Review the context preview dialog before sending auto-captured traffic.
* Review the [Privacy Modes](../privacy/privacy-modes.md) page for redaction patterns.

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

Transient network failures trigger automatic retries (max 6 attempts) with the standard stepped backoff (`500 / 1000 / 1500 / 2000 / 3000 / 4000 ms`). Each retry is recorded in the [AI Request Logger](../privacy/ai-request-logger.md) as a `RETRY` activity. After 5 consecutive failures the circuit breaker opens for 30 seconds before allowing a half-open probe.

## Related Pages

* [Backends Overview](overview.md)
* [Generic (OpenAI-compatible)](openai-compatible.md)
* [Troubleshooting](../reference/troubleshooting.md)
