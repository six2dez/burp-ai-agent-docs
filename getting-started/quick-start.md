# Quick Start

This walkthrough gets from installation to first AI analysis quickly.

## 1. Enable MCP (Optional but Recommended)

* Click **MCP** toggle in the top bar of the **AI Agent** tab.
* Verify indicator turns active.

![Screenshot: Top bar toggles](../.gitbook/assets/top-bar-toggles.png)

## 2. Select Agent Profile (Recommended)

Bundled profiles are installed automatically into `~/.burp-ai-agent/AGENTS/`.

To add custom profiles, place `*.md` files in that directory and refresh profiles in **AI Backend** settings.

## 3. Configure AI Backend

1. Open **AI Backend** tab in Settings.
2. Choose backend.
3. Set CLI command or HTTP URL/model fields.

{% hint style="warning" %}
If using cloud CLIs, required credentials must exist in the runtime environment where Burp is launched. GUI launchers often do not inherit shell env vars.
{% endhint %}

For backend-specific values, see [Backends Overview](../backends/overview.md).

![Screenshot: Backend selection](../.gitbook/assets/backend-selection.png)

## 4. Analyze a Request

1. Go to **Proxy -> HTTP History**.
2. Right-click a request.
3. Select **Extensions -> Burp AI Agent -> Find vulnerabilities**.

![Screenshot: Context menu on request](../.gitbook/assets/context-menu-request.png)

## 5. Review Response

A new chat session opens and streams AI analysis.

![Screenshot: Chat response](../.gitbook/assets/chat-response.png)

## 6. Enable Background Scanning (Advanced)

1. Toggle **Passive** ON in top bar.
2. Browse target traffic.
3. Review findings in passive scanner view and Burp issues.

<figure><img src="../.gitbook/assets/quick-start-passive-toggle.png" alt="Passive scanner toggle enabled in top bar"><figcaption></figcaption></figure>

## Next Steps

* [UI Tour](../user-guide/ui-tour.md)
* [Context Menus](../user-guide/context-menus.md)
* [Passive AI Scanner](../scanners/passive.md)
