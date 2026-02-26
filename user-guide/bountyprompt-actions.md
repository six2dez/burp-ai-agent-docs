# BountyPrompt Actions

BountyPrompt integration adds curated, tag-aware actions to the request/response context menu. It is optional and fully controlled from the **Prompt Templates** settings tab.

## What This Adds

* A dedicated **BountyPrompt** submenu in request/response context menus.
* Curated prompt loading from a local JSON directory.
* Selective context extraction using `[HTTP_*]` tags.
* Optional automatic Burp issue creation for prompts with `outputType = issue`.

## Enable and Configure

Open **Prompt Templates** in the bottom settings panel and configure:

* **Enable BountyPrompt actions**
* **Prompt directory**
* **Auto-create issues**
* **Issue confidence threshold**
* **Enabled prompt IDs**

Reference defaults are documented in [Settings Reference](../reference/settings-reference.md).

## Curated Prompt Set

Only curated IDs are loaded by design.

### Detection

* `API_Keys_Exposure_Detection`
* `CSRF_Vulnerability_Assessment`
* `Security_Headers_Analysis`
* `Vulnerable_Software_Detection`
* `Vulnerable_File_Upload_Endpoint_Detection`
* `Sensitive_Error_Messages_Detection`

### Recon

* `Extract_Endpoints`

### Advisory

* `Web_Attack_Suggestions`

## Tag-Aware Context Resolution

BountyPrompt JSON `userPrompt` values can include these tags:

| Tag | Injected Data |
| :--- | :--- |
| `[HTTP_Requests]` | Redacted raw HTTP requests |
| `[HTTP_Requests_Headers]` | Request headers only |
| `[HTTP_Requests_Parameters]` | Parsed request parameters |
| `[HTTP_Request_Body]` | Request body only |
| `[HTTP_Responses]` | Redacted raw HTTP responses |
| `[HTTP_Response_Headers]` | Response headers only |
| `[HTTP_Response_Body]` | Response body only |
| `[HTTP_Status_Code]` | Response status code |
| `[HTTP_Cookies]` | Request/response cookie lines |

Unknown `[HTTP_*]` tokens are removed at resolution time.

## Privacy and Determinism Behavior

BountyPrompt actions follow the same privacy controls as other actions:

* Redaction runs before tag resolution.
* STRICT/BALANCED/OFF policies apply to the selected fields.
* Determinism mode controls stable ordering and host anonymization consistency.

## Prompt Composition

Each action composes two parts:

1. **System Instructions** from the BountyPrompt JSON `systemPrompt`.
2. **User Task** from resolved `userPrompt` after tag substitution.

The resulting text is sent as a standard chat action through the selected backend.

## Issue Creation Rules

Issue creation is attempted only when all conditions are true:

1. Prompt `outputType` is `issue`.
2. **Auto-create issues** is enabled.
3. Parser extracts one or more findings.
4. Parsed confidence is greater than or equal to **Issue confidence threshold**.

Additional behavior:

* Findings containing `NONE` are treated as no finding.
* JSON outputs are parsed from direct JSON or fenced JSON blocks.
* Fallback parsing stores raw output as issue detail if valid JSON findings are not extracted.
* Issue names are prefixed as `[AI][BountyPrompt] ...`.
* Duplicate issues (same base URL and same issue name) are skipped.

## Menu and UI Behavior

* Menu appears as **BountyPrompt** under request/response actions.
* Entries are grouped in **Detection**, **Recon**, and **Advisory**.
* Entry label includes selected item count.
* If disabled or unresolved, submenu is shown disabled with tooltip guidance.

## Operational Limits

* Only curated IDs are loaded.
* Only IDs listed in **Enabled prompt IDs** are allowed.
* Per-tag context is truncated before model submission:
  * **Detection**: `2500` chars per chunk, `10000` chars per tag.
  * **Recon**: `3500` chars per chunk, `14000` chars per tag.
  * **Advisory**: `3000` chars per chunk, `12000` chars per tag.
* Up to 20 selected request/response items are attached to created issues.

## Troubleshooting

For menu visibility, loading errors, and issue creation diagnostics, see [Troubleshooting](../reference/troubleshooting.md).
