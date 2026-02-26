# First Run Checklist

Use this checklist after installation before starting a real assessment.

## Essential Setup

* [ ] **Extension loaded**: `AI Agent` tab is visible.
* [ ] **Backend selected**: backend set in **AI Backend** tab.
* [ ] **Backend configured**: command/URL/model/auth values are valid.
* [ ] **Backend healthy**: top status indicator shows active state.
* [ ] **Context menus available**: request right-click menu shows Burp AI Agent actions.

## MCP Server (Recommended)

* [ ] **MCP ON**: top-bar MCP toggle enabled.
* [ ] **Token recorded**: token available for external clients when needed.
* [ ] **Port free**: configured port (default `9876`) is not occupied.

## Privacy & Security

{% hint style="warning" %}
Default privacy mode is `OFF`. Choose `STRICT` or `BALANCED` before using cloud backends on sensitive targets.
{% endhint %}

* [ ] **Privacy mode set** intentionally (`STRICT`/`BALANCED`/`OFF`).
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
3. Select **Extensions -> Burp AI Agent -> Find vulnerabilities**.
4. Verify a chat session opens and response streams.

If any step fails, use [Troubleshooting](../reference/troubleshooting.md).

<figure><img src="../.gitbook/assets/first-run-context-menu.png" alt="Context menu showing Burp AI Agent actions during first-run verification"><figcaption></figcaption></figure>
