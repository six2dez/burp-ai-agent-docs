# Generic (OpenAI-compatible)

Use this backend to connect to any provider that implements the OpenAI-compatible Chat Completions API.

## Setup

1. Obtain the base URL from your provider.
2. Identify the model name you want to use.
3. Configure in Burp: **Burp AI Agent → AI Backend tab in the bottom settings panel**:

| Setting                     | Value                          |
| -------------------------- | ------------------------------ |
| **Preferred Backend**      | `Generic (OpenAI-compatible)`  |
| **Base URL**               | Provider base URL              |
| **Model**                  | Provider model identifier      |
| **API Key (Bearer)**       | *(optional)* if required       |
| **Extra Headers**          | *(optional)* additional headers |
| **Timeout (seconds)**      | Increase for large responses   |

### URL behavior

The extension builds the final endpoint as follows:

* If the Base URL ends with `/v1` or `/v4` (or any `/vN`), it appends `/chat/completions`.
* If the Base URL already ends with `/chat/completions`, it uses it as-is.
* Otherwise, it appends `/v1/chat/completions`.

Examples:

```
Base URL: https://api.example.com       -> https://api.example.com/v1/chat/completions
Base URL: https://api.example.com/v1    -> https://api.example.com/v1/chat/completions
Base URL: https://api.example.com/v4    -> https://api.example.com/v4/chat/completions
```

Headers are entered one per line:

```
X-Org: myorg
X-Project: myproj
```

## Notes

* The backend uses non-streaming responses by default.
* If the provider requires authentication, set **API Key (Bearer)** or provide the header manually.

## Troubleshooting

* **HTTP 401/403**: Check your API key and headers.
* **HTTP 404**: Verify the base URL and that the Chat Completions endpoint is supported.
* **Timeouts**: Increase the timeout value or use a smaller model.
