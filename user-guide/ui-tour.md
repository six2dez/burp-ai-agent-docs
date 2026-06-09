# UI Tour

This page explains each major area of the Custom AI Agent interface. (In Burp the extension loads under the display name **Custom AI Agent** — its entry in the **Extensions** list and the Suite tab title — to distinguish it from Burp's built-in "Burp AI" provider.)

## Top Bar

| Control              | Description                                             |
| -------------------- | ------------------------------------------------------- |
| **MCP toggle**       | Starts/stops MCP runtime.                               |
| **Passive toggle**   | Enables/disables passive scanner.                       |
| **Active toggle**    | Enables/disables active scanner.                        |
| **Backend selector** | Backend for new sessions (available only).              |
| **Status indicator** | Health state (`AI: OK`, `AI: Degraded`, `AI: Offline`). |

<figure><img src="../.gitbook/assets/Screenshot 2026-05-15 at 10.28.50.png" alt=""><figcaption></figcaption></figure>

## Chat Panel

* Session list and per-session metadata.
* Usage stats (messages, chars, token estimates).
* Markdown-rendered streaming responses.
* Input box (`Enter` send, `Shift+Enter` newline).
* **Tools** menu for MCP-assisted prompts.
* Context preview and cancellation controls.
* Markdown export per session.

## Settings Panel

The panel is docked at the bottom of the AI Agent tab and can be resized/collapsed.

All Settings tabs are built on a shared internal design system (consistent spacing/typography, one-line section descriptions, and collapsible sections on the dense scanner tabs). Colors are theme-aware tokens, so the UI re-themes automatically when Burp switches between light and dark — there are no hardcoded colors. Settings keys and persistence are unchanged, so previously saved configs load as-is.

| Tab | Purpose |
| :--- | :--- |
| **AI Backend** | Backend commands/URLs/models/auth and health testing. |
| **AI Passive Scanner** | Scope/rate/size controls, dedup/cache, context caps, findings. |
| **AI Active Scanner** | Concurrency, risk level, scan mode, queue controls, Collaborator. |
| **MCP Server** | Host/port/TLS/token/limits/unsafe-tool master switch. |
| **Burp Integration** | Redesigned MCP Tools panel: tools grouped extension-native (AI) vs generic (Montoya), each tagged store-build / full-build, with search/filter and per-group bulk toggles. |
| **Prompt Templates** | Built-in template editing and BountyPrompt controls. |
| **Custom Prompts** | Saved free-form prompt library: add/edit/duplicate/delete, ★ favorites (pinned to the top of the context menu), live search filter, JSON import/export, per-entry context-menu visibility tags. |
| **Privacy & Logging** | Privacy mode, determinism, salt, audit logging, AI request logger. |
| **AI Logger** | Real-time AI activity log with filters, trace correlation, and export. |
| **Help** | Quick docs and setup references. |

<figure><img src="../.gitbook/assets/ui-tour-settings-panel.png" alt="Settings panel tabs in the AI Agent UI"><figcaption></figcaption></figure>
<!-- TODO: refresh ui-tour-settings-panel.png — the settings panel now has 10 tabs (new "Custom Prompts" tab between Prompt Templates and Privacy & Logging). -->

<figure><img src="../.gitbook/assets/Screenshot 2026-05-15 at 10.29.37.png" alt=""><figcaption></figcaption></figure>

## Privacy Indicator

Visual pill indicates active mode (`STRICT`, `BALANCED`, `OFF`) so you can confirm policy before sending prompts. A second pill — the **Safety Indicator** — surfaces the active scanner posture as `OK` (green), `WARN` (yellow), or `RISK` (red), with a tooltip explaining which switches drove the level.

## Advisory Banner (SubtleNotice)

The **Privacy & Logging** and **MCP Server** settings tabs render advisory state through a single inline banner instead of stacked red labels. It auto-selects one of three levels based on the combined state of privacy mode, MCP exposure, **Enable Unsafe Tools**, and the active scanner:

| Level             | When it shows                                                                                     | Typical example                                                                                  |
| ----------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **INFO** (blue)   | Posture is unusual but not risky — worth noting before you click Save.                            | Privacy mode `OFF` with MCP running.                                                             |
| **WARN** (yellow) | A reversible combination that increases exposure if left in place.                                | Privacy `STRICT` with the active scanner on; external MCP without **Allowed Origins** populated. |
| **RISK** (red)    | A combination that should be deliberate — exposes traffic or tools outside the loopback boundary. | External MCP with **Enable Unsafe Tools** on; active scanner on with Privacy `OFF`.              |

The banner supports multi-line HTML wrapping inside the `GridBagLayout` rows, repaints automatically when you switch Burp's theme, and collapses cleanly when there is nothing to report — no dangling "Advisory:" label remains visible. See [Privacy Modes](../privacy/privacy-modes.md) and [MCP Overview](../mcp/overview.md) for the per-setting consequences the banner is summarizing.

## Context Preview Dialog

Every right-click action that auto-captures context (proxy item, scanner issue, site-map node, Repeater) opens a modal before anything leaves the plugin. The modal shows the action, current privacy mode, the exact prompt, and the exact redacted JSON that will be sent. Confirm with **Send** or abort with **Cancel**. See [Context Menus → Context Preview Dialog](context-menus.md#context-preview-dialog) for details.

## Keyboard Shortcuts

* `Cmd/Ctrl + N`: new session
* `Cmd/Ctrl + W`: delete session
* `Cmd/Ctrl + L`: clear chat
* `Cmd/Ctrl + E`: export chat
* `Esc`: toggle settings panel

## Findings Panels

Scanner dialogs expose findings, severity/confidence, and queue/runtime metrics.

## Next Steps

* [Context Menus](context-menus.md)
* [Chat & Sessions](chat-sessions.md)
* [Burp Integration](burp-integration.md)
