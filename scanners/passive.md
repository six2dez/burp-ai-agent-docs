# Passive AI Scanner

The passive scanner analyzes proxy traffic in the background and can create Burp issues automatically. It observes existing traffic only and does not send extra requests by itself.

## How It Works

1. Proxy traffic is captured as you browse.
2. Requests/responses are filtered (scope, MIME, size, stream patterns).
3. Local checks run before AI calls.
4. Dedup and prompt-result cache reduce repeated analysis.
5. Qualified items are sent to the selected backend.
6. Findings with confidence `>= 85%` can become `[AI Passive]` issues.

## Passive-to-Active Handoff

```mermaid
flowchart LR
    Traffic[Proxy traffic]
    Prefilter[Scope, MIME, size, stream filters]
    Local[Local checks]
    Dedup[Endpoint and fingerprint dedup]
    Cache{Prompt cache hit?}
    Hit[Reuse cached parsed findings]
    AI[Run AI analysis]
    Gate{Confidence >= 85% and severity gate?}
    Issue[Create [AI Passive] issue]
    Auto{Auto-Queue from Passive enabled?}
    Active[Queue in Active Scanner]

    Traffic --> Prefilter --> Local --> Dedup --> Cache
    Cache -->|Yes| Hit --> Gate
    Cache -->|No| AI --> Gate
    Gate -->|No| End[No issue]
    Gate -->|Yes| Issue --> Auto
    Auto -->|Yes| Active
    Auto -->|No| End2[Passive only]
```

## Configuration

### Core Controls

| Setting | Default | Description |
| :--- | :--- | :--- |
| **Enabled** | Off | Toggle in top bar or in the `AI Passive Scanner` settings tab. |
| **Rate Limit** | `5s` | Minimum delay between analysis requests (range: 1–60). |
| **Scope Only** | On | Analyze only in-scope targets. |
| **Max Size (KB)** | `96` | Maximum response size eligible for passive analysis (range: 16–1024). |
| **Min Severity** | `LOW` | Ignore findings below selected severity. |

### Token/Performance Controls

| Setting | Default | Description |
| :--- | :--- | :--- |
| **Endpoint dedup (min)** | `30` | Skip equivalent method/path analyses inside window. |
| **Response dedup (min)** | `30` | Skip repeated response fingerprints inside window. |
| **Prompt cache TTL (min)** | `30` | Reuse parsed results for identical prompts. |
| **Prompt cache entries** | `500` | Maximum prompt-result cache entries. |
| **Endpoint cache entries** | `5000` | Maximum endpoint dedup entries. |
| **Fingerprint cache entries** | `5000` | Maximum response-fingerprint dedup entries. |
| **Req body chars (AI)** | `2000` | Max request body chars in passive metadata. |
| **Resp body chars (AI)** | `4000` | Max response body chars in passive metadata. |
| **Max headers** | `40` | Max filtered headers in passive metadata. |
| **Max params** | `15` | Max request params in passive metadata. |
| **Req body chars (manual)** | `4000` | Max request body chars for manual context actions. |
| **Resp body chars (manual)** | `8000` | Max response body chars for manual context actions. |
| **Manual context JSON** | On (compact) | Compact JSON for context-menu payloads. |

{% hint style="tip" %}
If cloud cost is high, lower `Resp body chars (AI)`, `Max headers`, `Max params`, and `Max Size (KB)` before disabling passive scanning entirely.
{% endhint %}

![Screenshot: Passive scanner settings](../.gitbook/assets/passive-scanner.png)

## MIME Type Filtering

The scanner processes text-like content types:

* `text/html`
* `application/json`
* `application/javascript` / `text/javascript`
* `application/xml` / `text/xml`
* `text/plain`
* `unknown` (unrecognized textual responses)

Binary assets are skipped.

## Detection Rules (Local Checks)

### CSRF Token Detection

Patterns include: `csrf`, `xsrf`, `anti_csrf`, `csrfmiddlewaretoken`, `__requestverificationtoken`, `token`.

### Dangerous File Upload Extensions

Examples: `php`, `jsp`, `aspx`, `cgi`, `py`, `jar`, `war`, `exe`, `dll`.

### Authentication Header Detection

Examples: `Authorization`, `X-API-Key`, `X-Auth-Token`, `X-Access-Token`.

### Session Cookie Detection

Session-like cookie keys: `session`, `auth`, `token`, `sid`, `jwt`, `remember`.

### Header Injection Points

Header allowlist used for injection contexts:

* `Host`
* `Origin`
* `Referer`
* `X-Forwarded-Host`
* `X-Forwarded-For`
* `X-Host`
* `X-Original-Host`

## Token and Noise Reduction Pipeline

To reduce model spend while preserving useful evidence:

* Security-focused header filtering (noise headers are dropped).
* Parameter compaction with cache-busting key removal.
* Adaptive body compaction:
  * JSON array sampling,
  * HTML focus on head/forms/inline scripts,
  * bounded raw-text excerpts.
* Endpoint dedup + fingerprint dedup + prompt cache.

## Output

### Findings View

Open **AI Passive Scanner tab -> View findings** to inspect:

* timestamp,
* URL,
* title,
* severity,
* detail,
* reasoning (if model provided it),
* confidence.

### Issue Creation

Automatic issue creation requires all conditions:

* confidence `>= 85%`,
* severity passes `Min Severity`,
* finding is not duplicate-equivalent for same base URL + canonical name.

Issues are prefixed with `[AI Passive]`.

## Status Tracking

Passive runtime view includes:

* requests analyzed,
* issues found,
* last analysis time,
* queue size.

## Related Pages

* [Active AI Scanner](active.md)
* [Settings Reference](../reference/settings-reference.md)
* [Troubleshooting](../reference/troubleshooting.md)
