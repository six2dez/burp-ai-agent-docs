# Redaction Pipeline

The redaction pipeline is the core privacy mechanism of the extension. It transforms raw Burp data into privacy-safe prompts before any data is sent to an AI backend or MCP client.

## Design Goals

*   **Prevent data leakage**: Ensure sensitive information (tokens, cookies, hostnames) does not reach unauthorized AI providers.
*   **Maintain analytical value**: Redacted data should still be useful for security analysis (e.g., the AI can see there *is* a Bearer token, even though the value is hidden).
*   **Deterministic output**: When determinism mode is enabled, the same input always produces the same redacted output, enabling reproducible audit trails.

## Privacy Modes

| Mode | Cookies | Auth Tokens | Hostnames |
| :--- | :--- | :--- | :--- |
| **STRICT** | Stripped | Redacted | Anonymized |
| **BALANCED** | Stripped | Redacted | Preserved |
| **OFF** | Preserved | Preserved | Preserved |

## Redaction Steps

### 1. Cookie Stripping
Removes `Cookie:` and `Set-Cookie:` header values entirely.
*   **Applies in**: STRICT, BALANCED
*   **Skipped in**: OFF

### 2. Auth Token Redaction
Redacts values from authentication-related headers:

| Header | Replacement |
| :--- | :--- |
| `Authorization` | `Bearer [REDACTED]` or `Basic [REDACTED]` |
| `Proxy-Authorization` | `[REDACTED]` |
| `X-API-Key` | `[REDACTED]` |
| `API-Key` | `[REDACTED]` |

JWT tokens (matching `eyJ...` patterns) are replaced with `[JWT_REDACTED]`.

*   **Applies in**: STRICT, BALANCED
*   **Skipped in**: OFF

### 3. Host Anonymization
Replaces hostnames with deterministic pseudonyms using HMAC-style hashing:

```
pseudonym = "host-" + SHA256(salt + ":" + hostname)[0:6] + ".local"
```

Example: `bank-of-america.com` → `host-a3f2c1.local`

The mapping is **stable** within a session: the same hostname always maps to the same pseudonym, so the AI can correlate findings across multiple requests to the same host.

A **reverse mapping** is maintained internally so the extension can translate pseudonyms back to real hosts when executing MCP tool actions.

*   **Applies in**: STRICT
*   **Skipped in**: BALANCED, OFF

## Important Notes

*   **Active Scanner bypass**: The active scanner always sends traffic to real targets. Redaction only applies to the *prompt* sent to the AI, not to the payloads sent to the target application.
*   **MCP integration**: Data returned by MCP tools is also filtered through the redaction pipeline before being sent to external clients.
*   **Salt rotation**: Rotate the host anonymization salt between engagements to prevent cross-engagement correlation.

## Testing

The redaction pipeline has dedicated unit tests in `RedactionTest.kt` that verify:
*   Cookie stripping behavior across all privacy modes.
*   Token redaction patterns (Bearer, Basic, JWT, API keys).
*   Host anonymization stability (deterministic output with the same salt).
*   Edge cases (empty headers, malformed tokens, URLs in various formats).
