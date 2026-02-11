# Settings Reference

This page documents every configurable setting in the extension, organized by section. Settings are grouped into tabs in the bottom settings panel of the AI Agent UI.

## AI Backend

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Preferred Backend** | Dropdown | `codex-cli` | Backend used for new chat sessions. The list is filtered to backends that are available on this machine. |
| **Codex CLI Command** | Text | `codex chat` | Shell command to launch the Codex backend (Windows npm shim example: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\codex.cmd`). |
| **Gemini CLI Command** | Text | `gemini --output-format text --model gemini-2.5-flash --yolo` | Shell command to launch the Gemini backend (Windows npm shim example: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\gemini.cmd`). |
| **Claude CLI Command** | Text | `claude` | Shell command to launch the Claude backend (Windows npm shim example: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\claude.cmd`). |
| **OpenCode CLI Command** | Text | `opencode` | Shell command to launch the OpenCode backend (Windows npm shim example: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\opencode.cmd`). |
| **Ollama URL** | Text | `http://127.0.0.1:11434` | Base URL for the Ollama HTTP API. |
| **Ollama Model** | Text | `llama3.1` | Model name to use with Ollama. |
| **Ollama API Key** | Text | *(empty)* | Optional `Authorization: Bearer` token for Ollama-compatible servers. |
| **Ollama Extra Headers** | Multiline | *(empty)* | Extra headers, one per line (`Header: value`). |
| **Ollama Auto-Start** | Toggle | On | Automatically start `ollama serve` when backend is selected. |
| **Ollama Serve Command** | Text | `ollama serve` | Custom command to start the Ollama server. |
| **Ollama Timeout** | Number | `120` | Timeout in seconds for Ollama requests (range: 30–3600). |
| **Ollama Context Window** | Number | `8192` | Context window sent to Ollama (range: 2048–128000). |
| **LM Studio URL** | Text | `http://127.0.0.1:1234` | Base URL for the LM Studio HTTP API. |
| **LM Studio Model** | Text | `lmstudio` | Model identifier for LM Studio. |
| **LM Studio API Key** | Text | *(empty)* | Optional `Authorization: Bearer` token for OpenAI-compatible servers. |
| **LM Studio Extra Headers** | Multiline | *(empty)* | Extra headers, one per line (`Header: value`). |
| **LM Studio Auto-Start** | Toggle | On | Automatically start LM Studio server. |
| **LM Studio Server Command** | Text | `lms server start` | Custom command to launch LM Studio server. |
| **LM Studio Timeout** | Number | `120` | Timeout in seconds for LM Studio requests (range: 30–3600). |
| **OpenAI-Compatible URL** | Text | *(empty)* | Base URL for an OpenAI-compatible provider (include `/v1` or `/vN` if required). |
| **OpenAI-Compatible Model** | Text | *(empty)* | Model identifier sent in requests. |
| **OpenAI-Compatible API Key** | Text | *(empty)* | Optional `Authorization: Bearer` token. |
| **OpenAI-Compatible Extra Headers** | Multiline | *(empty)* | Extra headers, one per line (`Header: value`). |
| **OpenAI-Compatible Timeout** | Number | `120` | Timeout in seconds for OpenAI-compatible requests (range: 30–3600). |
| **Auto-Restart** | Toggle | On | Automatically restart a crashed CLI backend. |
| **Agent Profile** | Dropdown | `pentester` | System instruction profile. Options are loaded from `~/.burp-ai-agent/AGENTS/*.md` (use **Refresh** to reload). See [Agent Profiles](../user-guide/agent-profiles.md). |

## Privacy & Logging

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Privacy Mode** | Dropdown | `OFF` | Controls data redaction. Options: `STRICT`, `BALANCED`, `OFF`. See [Privacy Modes](../privacy/privacy-modes.md). |
| **Determinism Mode** | Toggle | Off | Stabilizes context ordering for reproducible prompt bundles. |
| **Host Anonymization Salt** | Text | *(auto-generated)* | Secret string used to generate stable host pseudonyms in STRICT mode. Rotate per engagement. |
| **Audit Logging** | Toggle | Off | Enables JSONL audit logging of all AI interactions. Logs saved to `~/.burp-ai-agent/audit.jsonl`. |

## MCP Server

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Enable MCP** | Toggle | On | Start the MCP SSE server on extension load. |
| **Host** | Text | `127.0.0.1` | Bind address. Use `127.0.0.1` for local-only access. |
| **Port** | Number | `9876` | TCP port for the SSE server. |
| **External Access** | Toggle | Off | Allow connections from non-loopback addresses. **Requires TLS.** |
| **STDIO Bridge** | Toggle | Off | Enable stdin/stdout MCP transport in addition to SSE. |
| **Token** | Text | *(auto-generated)* | Bearer token for external access. Localhost access does not require a token. |
| **TLS Enabled** | Toggle | Off | Enable HTTPS for the MCP server. |
| **Auto-Generate Certificate** | Toggle | On | Generate a self-signed PKCS12 keystore automatically. Stored at `~/.burp-ai-agent/certs/mcp-keystore.p12`. |
| **Keystore Path** | Text | *(auto-populated)* | Path to a custom PKCS12 keystore for TLS. |
| **Keystore Password** | Text | *(auto-generated)* | Password for the custom keystore. |
| **Max Concurrent Requests** | Number | `4` | Maximum parallel MCP tool calls (range: 1–64). |
| **Max Body Bytes** | Number | `2097152` | Maximum response body size returned by MCP tools (range: 256 KB – 100 MB). |
| **Enable Unsafe Tools** | Toggle | Off | Master switch for all tools marked as "unsafe". |

## Burp Integration

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Tool Toggles** | Checkboxes | *(varies)* | Enable/disable individual MCP tools by category. See [Tools Reference](../mcp/tools-reference.md). |

## Passive AI Scanner

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Enabled** | Toggle | Off | Enable background passive analysis of proxy traffic. |
| **Rate Limit** | Number | `5` | Minimum seconds between analysis requests (range: 1–60). |
| **Scope Only** | Toggle | On | Only analyze requests to in-scope targets. |
| **Max Size (KB)** | Number | `96` | Maximum request+response size sent for analysis (range: 16–1024 KB). |
| **Min Severity** | Dropdown | `LOW` | Ignore findings below this severity level. Options: `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`. |

## Active AI Scanner

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Enabled** | Toggle | Off | Enable active vulnerability testing. |
| **Max Concurrent Scans** | Number | `3` | Maximum parallel active scans (range: 1–10). |
| **Max Payloads per Point** | Number | `10` | Maximum payload variations per injection point (range: 1–50). |
| **Timeout** | Number | `30` | Timeout in seconds per scan request (range: 5–120). |
| **Request Delay** | Number | `100` | Delay in milliseconds between requests (range: 0–5000). |
| **Max Risk Level** | Dropdown | `SAFE` | Maximum payload risk level. Options: `SAFE`, `MODERATE`, `DANGEROUS`. |
| **Scope Only** | Toggle | On | Only scan in-scope targets. |
| **Scan Mode** | Dropdown | `FULL` | Vulnerability class selection strategy. Options: `BUG_BOUNTY`, `PENTEST`, `FULL`. |
| **Auto-Queue from Passive** | Toggle | On | Automatically queue high-confidence passive findings for active verification. |
| **Use Collaborator (OAST)** | Toggle | Off | Enable out-of-band testing with Burp Collaborator payloads. |

## Prompt Templates

| Setting | Type | Description |
| :--- | :--- | :--- |
| **Find Vulnerabilities** | Text area | Default prompt for the "Find vulnerabilities" context menu action. |
| **Analyze this request** | Text area | Default prompt for endpoint summary. |
| **Explain JS** | Text area | Default prompt for JavaScript behavior analysis. |
| **Access Control** | Text area | Default prompt for authorization test plan generation. |
| **Login Sequence** | Text area | Default prompt for login flow extraction. |
| **Analyze this Issue** | Text area | Default prompt for scanner issue analysis. |
| **Generate PoC & Validate** | Text area | Default prompt for proof-of-concept generation. |
| **Impact & Severity** | Text area | Default prompt for impact and severity assessment. |
| **Full Report** | Text area | Default prompt for complete vulnerability report generation. |

See [Prompt Defaults](prompt-defaults.md) for the full default text of each template.

## File Locations

The extension stores data in `~/.burp-ai-agent/`. All subdirectories are created automatically on startup.

| File/Directory | Purpose |
| :--- | :--- |
| `~/.burp-ai-agent/audit.jsonl` | Audit log entries (JSONL format with SHA-256 hashing). Created when audit logging is enabled. |
| `~/.burp-ai-agent/bundles/` | Prompt bundle JSON files (used in determinism mode for reproducible analysis). |
| `~/.burp-ai-agent/contexts/` | Context snapshot JSON files (SHA-256 indexed). |
| `~/.burp-ai-agent/certs/` | Auto-generated TLS certificates (`mcp-keystore.p12`). |
| `~/.burp-ai-agent/backends/` | Drop-in backend JAR files. Place custom `BackendFactory` JARs here for ServiceLoader discovery. See [Adding a Backend](../developer/adding-backend.md). |
| `~/.burp-ai-agent/AGENTS/` | Agent profile Markdown files. See [Agent Profiles](../user-guide/agent-profiles.md). |
| `~/.burp-ai-agent/AGENTS/default` | Text file containing the active profile name (e.g., `pentester.md`). |


## Runtime Defaults

Some operational limits are intentionally runtime constants (not direct UI fields):

- **Active scanner queue max**: `2000` targets (`ACTIVE_SCAN_MAX_QUEUE_SIZE`).
- **Active dedup window**: `1 hour` (`DEDUP_WINDOW_MS = 3,600,000`).
- **Passive analysis wait timeout**: `90s` (`PASSIVE_SCAN_TIMEOUT_MS`).
- **HTTP backend conversation history cap**: `20` messages (`MAX_HISTORY_MESSAGES`).
- **CLI history cap**: `10` messages or `20,000` characters (whichever is reached first).
- **Large prompt threshold for Claude CLI fallback path**: `32,000` characters.

### Additional backend setting notes

- **Ollama Timeout**: Default `120` seconds (range `30–3600`).
- **Ollama Context Window**: Default `8192` (range `2048–128000`).
