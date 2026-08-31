# First Run Checklist

Use this checklist after installation before starting a real assessment.

## Essential Setup

* [ ] **Extension loaded**: `Custom AI Agent` tab is visible.
* [ ] **Burp AI prerequisite checked** — if you plan to use the built-in **Burp AI** backend, open Burp's **Settings → Burp AI** and confirm *Use AI for extensions* is **ON** (Burp Pro only). Without this, the backend stays `Offline` and cannot be selected. See [Burp AI (Built-in)](../backends/burp-ai.md).
* [ ] **Backend selected**: backend set in **AI Backend** tab.
* [ ] **Backend configured**: command/URL/model/auth values are valid (or leave blank when using Burp AI built-in).
* [ ] **Backend healthy**: top status indicator shows active state.
* [ ] **Context menus available**: request right-click menu shows Custom AI Agent actions.

## MCP Server (Recommended)

* [ ] **MCP ON**: top-bar MCP toggle enabled.
* [ ] **Token recorded**: token available for external clients when needed.
* [ ] **Port free**: configured port (default `9876`) is not occupied.

## Privacy & Security

{% hint style="info" %}
Default privacy mode is `BALANCED` (recognized cookie/token carriers sanitized, hosts preserved). Switch to `STRICT` when covered host carriers also need anonymization, or `OFF` only for controlled local-model testing. Custom regexes still run under `OFF`; review [Privacy Modes](../privacy/privacy-modes.md) and the [known boundaries](../privacy/limitations.md#redaction-coverage-and-known-boundaries).
{% endhint %}

* [ ] **Privacy mode set** intentionally (`STRICT`/`BALANCED`/`OFF`).
* [ ] **Context preview dialog** confirmed at least once: right-click a proxy item, choose an AI action, and verify the modal shows privacy mode + prompt + redacted JSON before sending.
* [ ] **Audit logging** enabled if compliance traceability is needed.
* [ ] **Determinism** enabled if reproducibility is required.
* [ ] **Salt** rotated for new sensitive engagements.

## Scanners (Optional)

* [ ] **Passive scanner** configured with **Scope Only** ON.
* [ ] **Active scanner** only enabled when traffic is authorized.
* [ ] **Scope configured** in Burp Target before active checks.

## Verification Test

1. Browse through Burp Proxy.
2. Right-click a request in **Proxy -> HTTP History**.
3. Select **Extensions -> Custom AI Agent -> Find vulnerabilities**.
4. Verify a chat session opens and the completed response is rendered (current built-in backends deliver one chunk).

If any step fails, use [Troubleshooting](../reference/troubleshooting.md).

The exact menu contents depend on the invocation context: selecting request-side text can add **AI Scan on Selected Insertion Point**, and saved/favorite entries appear under **Custom prompts**.
