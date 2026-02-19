# UI Tour

This page describes each UI area and what it controls.

## Top Bar

The top bar provides quick access to core runtime controls:

| Control | Description |
| :--- | :--- |
| **MCP toggle** | Starts/stops the MCP server. Green indicates running state. |
| **Passive toggle** | Enables/disables background passive analysis of proxy traffic. |
| **Active toggle** | Enables/disables active scanning. See warnings in [Active Scanner](../scanners/active.md). |
| **Backend selector** | Chooses the AI backend for new sessions (available backends only). |
| **Status indicator** | Shows backend state such as `Ready`, `Starting`, `Running`, or `Crashed`. |

![Screenshot: Top bar](../.gitbook/assets/top-bar.png)

## Chat Panel

Main interaction area for AI conversations:

* **Session list**: Sidebar with title, timestamp, and backend metadata.
* **Usage stats**: Message and backend usage counters.
* **Chat area**: Markdown-rendered assistant output.
* **Input field**: `Enter` sends, `Shift+Enter` inserts new line.
* **Tools button**: Inserts MCP tool commands into input.
* **Context preview**: Shows the redacted context that will be sent.
* **Streaming**: Responses stream incrementally.
* **Cancel button**: Stops in-flight response.
* **Export**: Right-click session to export Markdown.

## Settings Panel

The settings panel is in the bottom area of the AI Agent tab. It can be resized or collapsed.

| Tab | What It Controls |
| :--- | :--- |
| **AI Backend** | Backend selection, CLI commands, HTTP URLs, model names, API keys, headers, auto-start, and timeout settings. |
| **AI Passive Scanner** | Rate limit, scope filtering, max size, minimum severity, findings view. |
| **AI Active Scanner** | Concurrency, payload depth, timeout, delay, risk level, scan mode, collaborator, queue controls. |
| **MCP Server** | Host, port, TLS, external access, STDIO bridge, token, limits, unsafe-tools switch. |
| **Burp Integration** | MCP tool toggles grouped by category with safe/unsafe gating. |
| **Prompt Templates** | Built-in request/issue prompt templates plus BountyPrompt integration controls. |
| **Privacy & Logging** | Privacy mode, determinism, host salt, audit logging. |
| **Help** | Quick links to docs and MCP setup instructions. |

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

**Open CLI buttons**: CLI-based backend settings include an **Open CLI** button to launch a terminal with the configured command and MCP tool access.

## Privacy Pill

Visual indicator of active privacy mode:

* **STRICT**: High-visibility indicator for heavy redaction.
* **BALANCED**: Moderate indicator.
* **OFF**: Warning-style indicator that raw context is being sent.

## Dependency Banner

If MCP is disabled, a banner is shown at the top of the tab reminding you to enable MCP for AI features.

## Keyboard Shortcuts

* **New session**: `Cmd/Ctrl + N`
* **Delete session**: `Cmd/Ctrl + W`
* **Clear chat**: `Cmd/Ctrl + L`
* **Export chat**: `Cmd/Ctrl + E`
* **Toggle settings panel**: `Esc`

## Findings Panel

When scanners are active, findings dialogs provide:

* Passive findings with severity/confidence/detail.
* Active findings with confirmation evidence.
* Status metrics (requests analyzed, issues found, queue size).
