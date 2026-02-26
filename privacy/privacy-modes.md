# Privacy Modes

Privacy mode controls what request/response data can leave Burp when the extension calls AI backends or returns MCP tool output.

Configure it in the **Privacy & Logging** tab in the Settings panel.

{% hint style="warning" %}
Default value is `OFF`. For real engagements, set privacy mode intentionally before sending any prompts.
{% endhint %}

## Mode Comparison

| Mode | Cookies | Auth Tokens | Hostnames | Typical Use |
| :--- | :--- | :--- | :--- | :--- |
| `STRICT` | Stripped | Redacted | Anonymized | Cloud backends with sensitive targets. |
| `BALANCED` | Stripped | Redacted | Preserved | Mixed workflows where host context is needed. |
| `OFF` | Preserved | Preserved | Preserved | Controlled internal testing only. |

## Decision Guide

```mermaid
flowchart TD
    Start[Choose privacy mode]
    Cloud{Using cloud backend?}
    Sensitive{Sensitive target or data?}
    NeedHost{Need real hostnames in model output?}

    Start --> Cloud
    Cloud -->|Yes| Sensitive
    Cloud -->|No| NeedHost

    Sensitive -->|Yes| Strict[Use STRICT]
    Sensitive -->|No| NeedHost

    NeedHost -->|Yes| Balanced[Use BALANCED]
    NeedHost -->|No| Strict

    Balanced --> Review[Review redaction behavior before sending prompts]
    Strict --> Review
    Review --> Off{Only in isolated internal test?}
    Off -->|Yes| OffMode[Use OFF temporarily]
    Off -->|No| Done[Keep selected mode]
```

## What Changes in Practice

### STRICT

* Hostnames are replaced with deterministic pseudonyms.
* Auth tokens are redacted.
* Cookies are stripped.

### BALANCED

* Hostnames stay visible.
* Auth tokens are redacted.
* Cookies are stripped.

### OFF

* Raw context is eligible for transmission.
* No automatic redaction is applied.

## Before/After Example

Raw request headers:

```http
Host: api.company.tld
Authorization: Bearer eyJhbGciOi...
Cookie: sessionid=abc123; csrftoken=xyz
```

`STRICT` output:

```http
Host: host-a3f2c1.local
Authorization: Bearer [REDACTED]
Cookie: [STRIPPED]
```

`BALANCED` output:

```http
Host: api.company.tld
Authorization: Bearer [REDACTED]
Cookie: [STRIPPED]
```

## Important Boundaries

{% hint style="danger" %}
Privacy mode does not prevent active scanner traffic from reaching the real target. It only controls prompt/tool data sent to AI clients.
{% endhint %}

* BountyPrompt tag resolution runs after redaction, so tags inherit current privacy policy.
* MCP tool responses are filtered by the same privacy mode.
* Determinism mode and salt handling affect reproducibility and anonymization stability.

## Related Pages

* [Redaction Pipeline](../developer/redaction-pipeline.md)
* [Determinism & Salt](determinism-salt.md)
* [Audit Logging](audit-logging.md)
