# Redaction Pipeline

The redaction pipeline transforms raw Burp context into privacy-safe payloads before data is sent to AI backends or external MCP clients.

## Pipeline Overview

```mermaid
flowchart LR
    Raw[Raw request/response context]
    Cookie[Cookie stripping]
    Token[Auth/token redaction]
    Host{Privacy mode}
    Strict[Host anonymization]\nSTRICT only
    Keep[Preserve hostnames]\nBALANCED and OFF
    Clean[Redaction output]

    Raw --> Cookie --> Token --> Host
    Host -->|STRICT| Strict --> Clean
    Host -->|BALANCED or OFF| Keep --> Clean
```

## Design Goals

* Prevent sensitive values from leaving Burp unexpectedly.
* Preserve enough structure for useful security analysis.
* Support deterministic outputs when reproducibility is required.

## Privacy Modes

| Mode | Cookies | Auth Tokens | Hostnames |
| :--- | :--- | :--- | :--- |
| **STRICT** | Stripped | Redacted | Anonymized |
| **BALANCED** | Stripped | Redacted | Preserved |
| **OFF** | Preserved | Preserved | Preserved |

## Redaction Steps

### 1. Cookie Stripping

Removes `Cookie:` and `Set-Cookie:` values.

* Applies to: `STRICT`, `BALANCED`
* Skipped in: `OFF`

### 2. Auth Token Redaction

Redacts values from authentication headers (`Authorization`, `Proxy-Authorization`, `X-API-Key`, `API-Key`) and JWT-like tokens.

* Applies to: `STRICT`, `BALANCED`
* Skipped in: `OFF`

### 3. Host Anonymization

In `STRICT`, hostnames are pseudonymized using a salt-based hash.

```text
pseudonym = "host-" + SHA256(salt + ":" + hostname)[0:6] + ".local"
```

The mapping is stable for the same salt and can be rotated per engagement.

## Important Notes

* Redaction applies to prompt/tool output data, not to active scanner network traffic.
* MCP tool results are also filtered by the active privacy policy.
* Rotate salt between engagements to reduce cross-project correlation.

## Testing

`RedactionTest.kt` covers cookie stripping, token redaction patterns, hostname anonymization stability, and malformed input edge cases.
