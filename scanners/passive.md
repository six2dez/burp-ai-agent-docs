# Passive AI Scanner

The passive scanner analyzes proxy traffic in the background and can create issues automatically. It observes HTTP responses as they pass through the Burp proxy without sending any additional requests.

## How It Works

1. As you browse the target application, all proxy traffic passes through the passive scanner.
2. Responses are filtered by MIME type, scope, and size limits.
3. Qualifying responses are queued for AI analysis.
4. The AI analyzes the request/response pair for security issues using pattern matching and contextual reasoning.
5. Findings with confidence >= 85% are automatically promoted to Burp issues with an `[AI Passive]` prefix.

## Configuration

| Setting           | Default   | Description                                                                                  |
| ----------------- | --------- | -------------------------------------------------------------------------------------------- |
| **Enabled**       | Off       | Toggle in the top bar or the AI Passive Scanner tab in the bottom settings panel.                                                     |
| **Rate Limit**    | 5 seconds | Minimum time between analysis requests (range: 1–60s). Prevents overwhelming the AI backend. |
| **Scope Only**    | On        | Only analyze requests to targets in Burp's scope.                                            |
| **Max Size (KB)** | 96 KB     | Maximum **response** size for analysis (range: 16–1024 KB).                                  |
| **Min Severity**  | LOW       | Ignore findings below this severity level (`LOW`, `MEDIUM`, `HIGH`, `CRITICAL`).             |

> **Trade-off**: Higher **Max Size** values (e.g., 500 KB) allow analysis of large JSON responses but significantly increase token costs for cloud backends or slow down local inference. The default of 96 KB covers most API endpoints.

![Screenshot: Passive scanner settings](../.gitbook/assets/passive-scanner.png)

## MIME Type Filtering

The scanner only processes responses with the following content types:

* `text/html`
* `application/json`
* `application/javascript` / `text/javascript`
* `application/xml` / `text/xml`
* `text/plain`
* `unknown` (unrecognized content types)

Binary content (images, fonts, video, etc.) is automatically skipped.

## Detection Rules

The passive scanner uses pattern-based detection for common security issues before sending context to the AI:

### CSRF Token Detection

Identifies missing or weak CSRF tokens by searching for patterns: `csrf`, `xsrf`, `anti_csrf`, `csrfmiddlewaretoken`, `__requestverificationtoken`, `token`

### Dangerous File Upload Extensions

Flags upload endpoints accepting dangerous extensions: `php`, `phtml`, `php5`, `asp`, `aspx`, `jsp`, `jspx`, `cgi`, `pl`, `py`, `rb`, `jar`, `war`, `ear`, `exe`, `dll`

### Authentication Header Detection

Identifies endpoints using authentication headers: `Authorization`, `X-API-Key`, `X-Auth-Token`, `X-Access-Token`

### Session Cookie Detection

Identifies session-related cookies by pattern matching: `session`, `auth`, `token`, `sid`, `jwt`, `remember`

### Header Injection Points

The scanner checks for injectable headers from a curated allowlist: `Host`, `Origin`, `Referer`, `X-Forwarded-Host`, `X-Forwarded-For`, `X-Host`, `X-Original-Host`

## Output

### Findings View

All passive analysis results are available via **AI Passive Scanner tab in the bottom settings panel → View findings**. Each finding includes:

* **Timestamp**: When the analysis occurred.
* **URL**: The analyzed endpoint.
* **Title**: Short description of the finding.
* **Severity**: `INFORMATION`, `LOW`, `MEDIUM`, `HIGH`, or `CRITICAL`.
* **Detail**: Full AI analysis with evidence.
* **Reasoning**: Model-provided analysis logic when present in scanner JSON output.
* **Confidence**: Percentage score (0–100%).

### Issue Creation

Findings are automatically promoted to Burp issues when:

* Confidence score >= **85%**
* Severity meets or exceeds the configured minimum.
* Issues are prefixed with `[AI Passive]` for easy identification in the Issues panel.

Additional behaviors:
* Issue details are sanitized to plain text (Markdown formatting removed).
* If the title can be mapped to a known vulnerability class, the issue name is normalized to `[AI Passive] <VULN_CLASS>`.
* Duplicate issues with the same name and base URL are consolidated (existing issue is kept).

## Status Tracking

The scanner tracks operational metrics:

* **Requests Analyzed**: Total number of request/response pairs processed.
* **Issues Found**: Total findings created.
* **Last Analysis Time**: Timestamp of the most recent analysis.
* **Queue Size**: Number of requests waiting for analysis.

## Passive-to-Active Pipeline

When **Auto-Queue from Passive** is enabled in the **Active AI Scanner** settings, the passive scanner feeds confirmed findings into the active scanner:

1. Passive scanner identifies a potential injection point or vulnerability pattern.
2. The finding is automatically queued for active testing.
3. The [Active AI Scanner](active.md) sends targeted payloads to verify the finding.
4. Confirmed vulnerabilities are reported as separate Burp issues.

This two-stage pipeline maximizes coverage while minimizing unnecessary active traffic.


## Reliability and Parsing Behavior

The passive scanner includes reliability and parsing hardening:

- Backend startup now uses bounded readiness polling instead of a fixed sleep.
- Response completion waiting uses latch synchronization instead of polling loops.
- AI JSON output parsing uses Jackson-based parsing for nested/escaped content support.
- Request metadata sent to AI is redaction-aware before prompt generation.
- Passive analysis completion has a bounded timeout window (90 seconds).

These changes are internal and keep the user-facing workflow unchanged.
