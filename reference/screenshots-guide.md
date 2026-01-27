# Screenshot Guide

All screenshots go in `GitBook/assets/screenshots/`. Use PNG format, capture at 2x resolution if on Retina. Crop to the relevant area — avoid full-desktop captures.

Below is the complete list of screenshots referenced across the documentation. Each entry shows the filename, what it must show, and which pages use it.

## Getting Started

| Filename | What to capture | Used in |
| :--- | :--- | :--- |
| `burp-extensions-add.png` | Burp's **Extensions > Installed > Add** dialog with the Burp AI Agent JAR selected. Extension type set to Java. | installation.md, README.md |
| `burp-ai-agent-tab.png` | The main Burp navigation bar showing the **Burp AI Agent** tab after successful load. | installation.md |
| `top-bar-toggles.png` | The extension top bar showing MCP toggle, Passive/Active scanner toggles, backend dropdown, and status indicator. | quick-start.md |
| `backend-selection.png` | The Settings panel with the backend dropdown expanded, showing all 6 backend options. | quick-start.md |
| `context-menu-request.png` | Right-click on a request in Proxy History showing **Extensions > Burp AI Agent** submenu with all request actions (Quick recon, Find vulnerabilities, etc.). | quick-start.md, context-menus.md, README.md |
| `chat-response.png` | A chat session with a completed AI response showing markdown rendering (headers, code blocks, bold text). | quick-start.md |
| `scanner-settings.png` | The Passive/Active scanner settings section in the Settings panel. | quick-start.md |
| `settings-overview.png` | Full Settings panel showing all sections collapsed or visible at a glance. | first-run-checklist.md |

## User Guide

| Filename | What to capture | Used in |
| :--- | :--- | :--- |
| `top-bar.png` | Close-up of the top bar with all toggles (MCP ON, Passive OFF, Active OFF), backend dropdown set to a backend, and status showing "Running". | ui-tour.md |
| `settings-panel.png` | The Settings panel expanded, showing at least the AI Backend and Privacy sections. | ui-tour.md |
| `chat-sessions.png` | The chat panel with multiple session tabs visible, showing session names and the active session. | chat-sessions.md |
| `tools-menu.png` | The "Tools" button dropdown in the chat panel, showing available MCP tools. | chat-sessions.md |
| `context-menu-issue.png` | Right-click on a scanner issue showing the **Extensions > Burp AI Agent** submenu with issue actions (Analyze, PoC, Impact, Full Report). | context-menus.md |
| `prompt-templates.png` | The Prompt Templates section in Settings, showing both Request Prompts and Issue Prompts text areas. | prompt-templates.md |
| `mcp-tool-toggles.png` | The MCP Server tool toggles section in Settings, showing safe/unsafe tool checkboxes. | burp-integration.md |

## Scanners

| Filename | What to capture | Used in |
| :--- | :--- | :--- |
| `passive-scanner.png` | The Passive AI Scanner settings: Enabled toggle, Rate Limit, Scope Only, Max Size, Min Severity. | passive.md |
| `active-scanner.png` | The Active AI Scanner settings: Enabled toggle, Max Concurrent, Payloads per Point, Risk Level, Scan Mode, Collaborator toggle. | active.md |

## Privacy & Logging

| Filename | What to capture | Used in |
| :--- | :--- | :--- |
| `privacy-settings.png` | The Privacy & Logging section in Settings: Privacy Mode dropdown, Host Salt field, Determinism toggle. | privacy-modes.md |
| `audit-logging.png` | The Audit Logging toggle and any visible log path indication in Settings. | audit-logging.md |
| `determinism-salt.png` | Close-up of the Determinism Mode toggle and Host Anonymization Salt field. | determinism-salt.md |

## Backends

| Filename | What to capture | Used in |
| :--- | :--- | :--- |
| `backend-settings.png` | The full AI Backend section in Settings showing the backend dropdown and at least one backend's configuration fields (e.g., Ollama URL + model). | overview.md |

## MCP Server

| Filename | What to capture | Used in |
| :--- | :--- | :--- |
| `mcp-settings.png` | The MCP Server section in Settings: Enable toggle, Host, Port, Token (partially visible), TLS options, STDIO toggle. | mcp/overview.md |

## README (repo root)

| Filename | What to capture | Used in |
| :--- | :--- | :--- |
| `main-tab.png` | The full Burp AI Agent tab showing the chat panel with a response and the settings panel or top bar visible. Hero screenshot for the repo. | README.md |

## Total: 20 screenshots

### Capture Tips

- Use Burp Suite Professional if possible (shows scanner-related features).
- Set the extension to a light theme for better readability.
- Blur or redact any real target data — use `example.com` or `testphp.vulnweb.com`.
- Ensure the backend status shows "Running" (green) in captures where the top bar is visible.
- For context menu captures, use a request to a recognizable URL so the menu context is clear.
