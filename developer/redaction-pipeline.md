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

Removes `Cookie:` and `Set-Cookie:` header values, replacing them with `[STRIPPED]`.

* Applies to: `STRICT`, `BALANCED`
* Skipped in: `OFF`

### 2. Auth Token Redaction

Redacts the values of authentication / session / secret headers and any inline token patterns found in the text.

**Header names matched (case-insensitive)**:

```
Authorization, Proxy-Authorization,
X-API-Key, API-Key, X-API-Secret, API-Secret, X-Client-Secret,
X-Auth-Token, Auth-Token, X-Access-Token, Access-Token,
X-Session-Token, Session-Token,
X-CSRF-Token, CSRF-Token, X-XSRF-Token
```

**Inline token patterns**:

* `Bearer <value>` → `Bearer [REDACTED]`
* `Basic <base64>` → `Basic [REDACTED]`
* JWT-shaped tokens (`eyJ…` with three base64url-encoded segments) → `[JWT_REDACTED]`

* Applies to: `STRICT`, `BALANCED`
* Skipped in: `OFF`

### 3. URL Query Parameter Redaction

Rewrites the value of sensitive query parameters in-place, preserving the key so URL structure remains analyzable.

```
?<key>=<redacted_value>
```

Keys matched (case-insensitive): `access_token`, `api_key`, `apikey`, `auth`, `token`, `key`, `secret`, `password`, `pwd`, `session`, `sid`, `code`.

* Applies to: `STRICT`, `BALANCED`
* Skipped in: `OFF`

### 4. Host Anonymization

In `STRICT`, hostnames are pseudonymized using a salt-based hash.

```text
pseudonym = "host-" + SHA256(salt + ":" + hostname)[0:6] + ".local"
```

The mapping is stable for the same salt and can be rotated per engagement.

## Important Notes

* Redaction applies to prompt/tool output data, not to active scanner network traffic.
* MCP tool results are also filtered by the active privacy policy.
* Rotate salt between engagements to reduce cross-project correlation.
* The regex set is a hand-curated allowlist of the most common patterns; it is **not** exhaustive. New patterns are added as they are observed. Teams with bespoke header conventions may want to pre-sanitize traffic at an upstream proxy before it reaches Burp.

## Testing

`RedactionTest.kt` covers:

* Cookie stripping (`Cookie`, `Set-Cookie`).
* Classic and custom auth headers (`Authorization`, `X-Auth-Token`, `X-Access-Token`, `X-CSRF-Token`, `X-Api-Secret`).
* Inline `Bearer`, `Basic`, and JWT patterns.
* URL query parameter redaction on common sensitive keys.
* Host anonymization stability across calls with the same salt.
* Salt rotation (`clearMappings`) invalidates old mappings.
* `OFF` mode preserves every pattern unchanged.
