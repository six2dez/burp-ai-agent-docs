# Insertion Point Scan

**AI Scan on Selected Insertion Point** is a right-click action that runs an active AI scan scoped to a single parameter, header, JSON field, XML element, or path-segment ID under the current text selection. It is the surgical counterpart to the broader **AI Active Scan**, which tests every extractable insertion point in the request.

Use it when you have a specific suspicion about one piece of input — a token in the URL path, a JSON field that looks reflected, an interesting header value — and you want to spend AI/active-scan budget only on that one point.

## How to Launch

1. Open the request in any editor that supports text selection: **Proxy → HTTP History**, **Repeater**, or any request editor that surfaces context menus.
2. Highlight the **value** of the insertion point you care about. The selection must be on the **request side** (not the response side) and must overlap a candidate.
3. Right-click → **Extensions → Custom AI Agent → AI Scan on Selected Insertion Point (`<type>`: `<name>`)**. The menu label echoes back the resolved insertion point so you can confirm the action understood your selection.
4. Pick the vulnerability classes to test from the multi-select dialog (same dialog used by **Targeted tests → Custom…**).
5. Confirm the queue-size warning prompt. The scan starts immediately and confirmed findings appear in **Target → Issues** with the `[AI] Confirmed` prefix.

If the selection does not intersect any candidate insertion point, or if the editor returns a response-side range, the menu item is hidden entirely — there is no "no match" placeholder. Clear the selection and pick the value of an actual parameter/header/field to make the item appear.

<!-- TODO: screenshot of the right-click context menu showing the entry "AI Scan on Selected Insertion Point (json field: user_id)" with the dynamic label visible. Save as .gitbook/assets/insertion-point-context-menu.png and replace this marker with:
![Screenshot: Insertion point context menu](../.gitbook/assets/insertion-point-context-menu.png)
-->


## Insertion Point Types

The extractor resolves the selection against the following types, in priority order:

| Type | Source | Notes |
| :--- | :--- | :--- |
| `URL_PARAM` | URL query parameters | Montoya exposes exact byte offsets via `ParsedHttpParameter.valueOffsets()`, so matching is precise. |
| `BODY_PARAM` | Form-urlencoded body parameters | Same exact-offset match as URL params. |
| `COOKIE` | Cookies parsed as parameters | Same exact-offset match. Selecting inside one cookie value resolves only that cookie, not the rest of the `Cookie:` line. |
| `HEADER` | Request headers | Matched by locating the `Header-Name: value` line in the raw request bytes. Duplicate identical lines resolve to the first occurrence. |
| `JSON_FIELD` | Top-level JSON object fields | Body must be valid JSON (or `Content-Type: application/json` with a regex fallback). Matched by substring of the value. Selecting only the key (not the value) is treated as a non-match — fall back to a full **AI Active Scan** if you need key-level coverage. |
| `XML_ELEMENT` | XML element text content | Body must be XML-shaped (`<` start) or `Content-Type: */xml`. Matched by substring inside `<element>…</element>`. |
| `PATH_SEGMENT` | Numeric or UUID/ObjectId-like segments in the URL path | Matches `^[0-9]+$`, 36-char UUIDs, and 24-char hex (Mongo ObjectId-style). Named as `path_id`. |

Match resolution is **first hit wins** in the order above. If your selection overlaps both a URL param and a path ID, the URL param is selected.

## Vulnerability Class Picker

Clicking the menu item opens a multi-select dialog listing every vulnerability class registered with the active scanner. The dialog supports Select All / Deselect All. The picker filters out:

* **Passive-only classes** (e.g. issues that only the passive scanner can detect from response patterns) — these are silently dropped before queueing.
* **Classes disabled by the current Scan Mode** — `BUG_BOUNTY`, `PENTEST`, and `FULL` each scope the catalog differently. See [Active AI Scanner → Scan Modes](../scanners/active.md) for the per-mode lists.

If after filtering no classes remain, you get a "No vulnerability classes selected" warning and nothing is queued.

<!-- TODO: screenshot of the vulnerability-class multi-select dialog (the same one used by Targeted Tests → Custom…). Save as .gitbook/assets/vuln-class-picker.png and replace this marker with:
![Screenshot: Vulnerability class picker](../.gitbook/assets/vuln-class-picker.png)
-->


## Priority and Queue Behavior

Each selected vulnerability class generates one queued target with **priority 60**, ahead of the default priority **50** used by both the broader manual **AI Active Scan** and the auto-queue from the passive scanner. In practice this means an insertion-point scan jumps over the existing manual/passive backlog and starts running as soon as the active scanner's worker slot frees up.

Other queue semantics:

* **Scope-only**: respects the **Scope Only** toggle on the **AI Active Scanner** settings tab. Out-of-scope requests are dropped before queueing — the dialog reports `0 target(s) queued` and points at scope or queue exhaustion as possible causes.
* **Queue limit**: targets are dropped when the active scan queue is at its cap (default `2000`, see [Settings Reference](../reference/settings-reference.md)).
* **Dedup bypass**: unlike the auto-queue path, this action is **not** rate-limited by the `processedTargets` window. Re-invoking on the same insertion point queues fresh targets each time — match the existing manual-scan behavior where the user is consciously asking for another pass.

The post-queue dialog reports `Queue size: <before> -> <after>` so you can see the inflation immediately. If `0 target(s) queued` is reported, the most likely causes are scope filtering, all classes filtered out by Scan Mode, or the queue already at its cap.

## When the Action Is Hidden

The menu item is omitted entirely (not greyed out) when **any** of the following hold:

* The context menu was opened outside a request editor (so there is no `messageEditorRequestResponse()`).
* The current selection lives on the **response** side of a request/response editor.
* The selection is empty (`selectionEnd <= selectionStart`).
* The selection range does not overlap any of the seven candidate insertion-point types.

Hiding instead of disabling keeps the menu compact: the action only advertises itself when it has something concrete to do.

## Safety and Boundaries

* This action **sends real traffic to the target** like every other Active AI Scanner action. Confirm scope and rate-limit settings before triggering it on third-party assets.
* All Active AI Scanner caps apply — concurrency limit, request delay, max risk level, Collaborator (OAST) toggle, and adaptive payloads — see [Active AI Scanner](../scanners/active.md).
* Privacy mode redaction applies to the AI-side prompts that drive payload selection and finding analysis. It does **not** scrub bytes actually delivered to the target server.
* Findings keep the same lifecycle as other Active AI Scanner output: queued → tested → confirmed via secondary signal → `[AI] Confirmed` issue under **Target → Issues**.

## Related Pages

* [Active AI Scanner](../scanners/active.md)
* [Context Menus](context-menus.md)
* [Settings Reference → Active AI Scanner](../reference/settings-reference.md#active-ai-scanner)
* [Troubleshooting](../reference/troubleshooting.md)
