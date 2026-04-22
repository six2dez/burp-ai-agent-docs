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
| **Preferred Backend** | `Generic (OpenAI-compatible)` |
| **Base URL** | provider URL |
| **Model** | provider model id |
| **API Key (Bearer)** | optional |
| **Extra Headers** | optional (`Header: value`) |
| **Timeout (seconds)** | increase for heavy prompts |

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

The extension sets `max_tokens` automatically per request type to ensure complete responses:

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

If a request fails due to a transient network error, the extension retries automatically with exponential backoff up to 6 attempts. Each retry is logged in the [AI Request Logger](../privacy/ai-request-logger.md).

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
