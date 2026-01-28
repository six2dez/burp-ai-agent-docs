# First Run Checklist

Use this checklist after installation to ensure everything is configured correctly before your first assessment.

## Essential Setup

* [ ] **Extension loaded**: The "AI Agent" tab appears in the main Burp navigation bar.
* [ ] **Backend selected**: A backend is chosen in **Settings → AI Backend** (e.g., Ollama, Gemini, Claude, Codex).
* [ ] **Backend configured**: The CLI command or HTTP URL is correct and the backend is reachable.
* [ ] **Backend running**: The status indicator in the top bar shows "Running" (green).
* [ ] **Context menus available**: Right-clicking a request in Proxy History shows "Burp AI Agent" actions under Extensions.

## MCP Server (Recommended)

* [ ] **MCP toggle ON**: The MCP toggle in the top bar is enabled and the status shows active.
* [ ] **Token noted**: You have copied the MCP token from **Settings → MCP Server** (needed for external clients).
* [ ] **Port accessible**: The configured port (default: 9876) is not blocked by another application.

## Privacy & Security

* [ ] **Privacy mode set**: Choose `STRICT` for sensitive engagements, `BALANCED` for general use, or `OFF` for internal testing.
* [ ] **Audit logging** (optional): Enable in **Settings → Privacy & Logging** for compliance-sensitive assessments.
* [ ] **Determinism** (optional): Enable if you need reproducible prompt bundles for audit trails.
* [ ] **Host salt** (optional): Set a unique salt for each engagement when using STRICT mode.

## Scanners (Optional)

* [ ] **Passive scanner**: Enable via the top bar toggle. Set **Scope Only** to ON and configure the rate limit.
* [ ] **Active scanner**: Enable only when ready to send traffic. Start with `SAFE` risk level. Ensure **Scope Only** is ON.
* [ ] **Scope configured**: Your target is added to Burp's **Target → Scope** before enabling scanners.

## Verification Test

1. Browse to any website through Burp Proxy.
2. Right-click a request in **Proxy → HTTP History**.
3. Select **Extensions → Burp AI Agent → Quick recon**.
4. Verify that a chat session opens and the AI returns a response.

If any step fails, check the [Troubleshooting](../reference/troubleshooting.md) guide.

![Screenshot: Settings overview](../.gitbook/assets/settings-overview.png)
