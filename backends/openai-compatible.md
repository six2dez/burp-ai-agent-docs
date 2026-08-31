# Generic (OpenAI-compatible)

Use this backend for any provider exposing an OpenAI-compatible Chat Completions API.

## Requirements

* Provider base URL.
* Model identifier.
* Optional API key/headers depending on provider.

## Setup

1. Get provider URL and model name.
2. Configure fields in **AI Backend** settings tab.
3. Validate with **Test connection**.

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `openai-compatible` |
| **Base URL** | provider URL |
| **Model** | provider model id |
| **API Key (Bearer)** | optional |
| **Extra Headers** | optional (`Header: value`) |
| **Timeout (seconds)** | `120` by default (accepted range: 30-3600) |

### URL Behavior

Final endpoint resolution:

* Base URL ends with `/vN` -> append `/chat/completions`.
* Base URL already ends with `/chat/completions` -> use as-is.
* Otherwise -> append `/v1/chat/completions`.

Examples:

```text
https://api.example.com    -> https://api.example.com/v1/chat/completions
https://api.example.com/v1 -> https://api.example.com/v1/chat/completions
https://api.example.com/v4 -> https://api.example.com/v4/chat/completions
```

Headers example:

```text
X-Org: myorg
X-Project: myproj
```

## Output Token Limits

The extension sets `max_tokens` per request type to bound output and reduce avoidable truncation:

| Request Type | `max_tokens` |
| :--- | :--- |
| **Chat** | 4096 |
| **Scanner (single request)** | 2048 |
| **Scanner (batch analysis)** | 4096 |
| **Payload generation** | 1024 |

## Troubleshooting

{% hint style="tip" %}
* `401/403`: verify auth credentials and headers.
* `404`: verify provider supports chat completions at resolved path.
* Timeout: increase timeout or choose smaller/faster model.
{% endhint %}

## Retry Behavior

If a request throws a classified transient transport exception, the extension makes up to 6 total attempts with five possible delays (500/1000/1500/2000/3000 ms). The connection's circuit breaker opens after 5 recorded transient failures and allows one half-open model request after 30 seconds. HTTP error responses are not retried inside the same call, although 429/5xx responses count toward the breaker. Each actual retry is logged in the [AI Request Logger](../privacy/ai-request-logger.md). See [Backends Overview → Retry Behavior](overview.md#retry-behavior).

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
