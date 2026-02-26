# Context Menus

The extension adds actions to Burp right-click menus in tools where HTTP traffic or scanner findings are selected.

## Where Context Menus Appear

* **Proxy -> HTTP History**: Right-click any request.
* **Repeater**: Right-click the request or response.
* **Site Map**: Right-click any entry.
* **Scanner -> Issues** (Pro only): Right-click any finding.

## Request Actions

When you right-click HTTP request/response items, these actions are available under **Extensions -> Burp AI Agent**:

| Action | Description |
| :--- | :--- |
| **AI Passive Scan** | Queues selected request(s) for passive AI analysis in the background. |
| **AI Active Scan** | Queues selected request(s) for active testing. Sends traffic to the target. |
| **Targeted tests** | Submenu for focused active checks (SQLi, XSS, SSRF, IDOR, etc.). Sends traffic to the target. |
| **BountyPrompt** | Curated submenu of tag-aware prompts loaded from JSON files. Uses selective context extraction before model invocation. |
| **Find vulnerabilities** | Broad security analysis for injection, auth/access, disclosure, and misconfiguration classes. |
| **Analyze this request** | Compact endpoint summary (method, route, auth, params, response type, security notes). |
| **Explain JS** | JavaScript behavior and risk analysis. |
| **Access control** | Authorization test plan for horizontal/vertical privilege escalation paths. |
| **Login sequence** | Login flow extraction and replay guidance from observed traffic. |

![Screenshot: Request menu](../.gitbook/assets/context-menu-request.png)

## BountyPrompt Submenu

The **BountyPrompt** submenu is grouped by category:

* **Detection**: API key exposure, CSRF assessment, security headers, vulnerable software, sensitive errors, vulnerable file upload endpoints.
* **Recon**: Endpoint extraction from observed traffic.
* **Advisory**: Suggested follow-up attack vectors.

Menu entries show the selected item count, for example `Security Headers Analysis (3)`.

If BountyPrompt integration is disabled or no curated prompts are loadable from the configured directory, the submenu appears disabled with a tooltip.

## Targeted Tests Submenu

Examples:

* SQLi
* XSS (Reflected/Stored/DOM)
* SSRF
* IDOR / BOLA
* Path Traversal / LFI
* Command Injection
* SSTI
* XXE
* Open Redirect

## Issue Actions (Burp Pro)

When you right-click scanner issues, these actions are available:

| Action | Description |
| :--- | :--- |
| **Analyze this issue** | Detailed analysis with root cause, evidence, and validation flow. |
| **Generate PoC & validate** | Step-by-step proof-of-concept with expected responses and success criteria. |
| **Impact & severity** | CIA impact, exploitability, business risk, and CVSS-style reasoning. |
| **Full report** | Structured vulnerability report for delivery. |

![Screenshot: Issue menu](../.gitbook/assets/context-menu-issue.png)

## Execution Flow

1. Select one or more requests/responses or scanner issues.
2. Trigger a context menu action.
3. The extension collects selection context.
4. Privacy mode redaction is applied.
5. Context payload size controls are applied (manual request/response body truncation + optional compact JSON serialization).
6. Action prompt/template is composed.
7. The prompt is sent to the selected AI backend.
8. Response streams into a chat session.

For BountyPrompt actions, tag resolution runs after redaction and before prompt composition.

## Context Size Controls for Manual Actions

The **AI Passive Scanner** settings tab includes manual context controls that also affect context-menu actions:

* **Req body chars (manual)** and **Resp body chars (manual)** bound the request/response body content captured for prompt context.
* **Manual context JSON** toggles compact vs pretty JSON serialization for the context envelope.

These controls reduce token usage for non-local models while keeping request/response headers and core metadata intact.

## Multiple Selections

You can select multiple requests in Proxy History or Site Map and run actions across all selected items. This is useful for:

* Batch passive scanning.
* Batch queuing for active scanning.
* Running the same BountyPrompt action across related endpoints.

## Active Scan Safety and Queue Visibility

For active scan actions, the context flow enforces safety and visibility:

* Confirmation dialogs before queueing active tests.
* Target validation before enqueue.
* Queue state visibility (current and maximum queue size).
* Explicit warning when no target is queued (filtered target or full queue).

## Next Steps

* [Chat & Sessions](chat-sessions.md)
* [Passive AI Scanner](../scanners/passive.md)
* [Active AI Scanner](../scanners/active.md)
