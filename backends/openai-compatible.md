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

## Troubleshooting

{% hint style="tip" %}
* `401/403`: verify auth credentials and headers.
* `404`: verify provider supports chat completions at resolved path.
* Timeout: increase timeout or choose smaller/faster model.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
