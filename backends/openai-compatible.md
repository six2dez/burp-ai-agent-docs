# Generic (OpenAI-compatible)

Use this backend to connect to any provider that implements the OpenAI-compatible Chat Completions API (`/v1/chat/completions`).

## Setup

1. Obtain the base URL from your provider.
2. Identify the model name you want to use.
3. Configure in Burp: **Burp AI Agent → Settings → AI Backend**:

| Setting                     | Value                          |
| -------------------------- | ------------------------------ |
| **Preferred Backend**      | `Generic (OpenAI-compatible)`  |
| **Base URL**               | Provider base URL              |
| **Model**                  | Provider model identifier      |
| **API Key (Bearer)**       | *(optional)* if required       |
| **Extra Headers**          | *(optional)* additional headers |
| **Timeout (seconds)**      | Increase for large responses   |

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
* **HTTP 404**: Verify the base URL and that `/v1/chat/completions` is supported.
* **Timeouts**: Increase the timeout value or use a smaller model.
