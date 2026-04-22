# Settings Reference

This page documents configurable settings in the extension, organized by settings tab.

For tuning model spend, also see [Token Usage & Cost Management](../user-guide/token-management.md).

## AI Backend

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Preferred Backend** | Dropdown | `codex-cli` | Backend used for new chat sessions. All registered backends are shown regardless of CLI availability; errors are reported at usage time via health check. |
| **Codex CLI Command** | Text | `codex chat` | Shell command used to launch Codex CLI. |
| **Gemini CLI Command** | Text | `gemini --output-format text --model gemini-2.5-flash --yolo` | Shell command used to launch Gemini CLI. |
| **Claude CLI Command** | Text | `claude` | Shell command used to launch Claude CLI. |
| **Copilot CLI Command** | Text | `copilot` | Shell command used to launch Copilot CLI. |
| **OpenCode CLI Command** | Text | `opencode` | Shell command used to launch OpenCode CLI. |
| **Ollama URL** | Text | `http://127.0.0.1:11434` | Base URL for Ollama API. |
| **Ollama Model** | Text | `llama3.1` | Model identifier sent to Ollama. |
| **Ollama API Key** | Text | *(empty)* | Optional bearer token for Ollama-compatible servers. |
| **Ollama Extra Headers** | Multiline | *(empty)* | Extra headers, one per line (`Header: value`). |
| **Ollama Auto-Start** | Toggle | On | Auto-start `ollama serve` when backend is selected. |
| **Ollama Serve Command** | Text | `ollama serve` | Command used for Ollama server startup. |
| **Ollama Timeout** | Number | `120` | Timeout in seconds (range: 30-3600). |
| **Ollama Context Window** | Number | `8192` | Context window size (range: 2048-128000). |
| **LM Studio URL** | Text | `http://127.0.0.1:1234` | Base URL for LM Studio API. |
| **LM Studio Model** | Text | `lmstudio` | Model identifier sent to LM Studio. |
| **LM Studio API Key** | Text | *(empty)* | Optional bearer token for OpenAI-compatible servers. |
| **LM Studio Extra Headers** | Multiline | *(empty)* | Extra headers, one per line (`Header: value`). |
| **LM Studio Auto-Start** | Toggle | On | Auto-start LM Studio server command. |
| **LM Studio Server Command** | Text | `lms server start` | Command used for LM Studio startup. |
| **LM Studio Timeout** | Number | `120` | Timeout in seconds (range: 30-3600). |
| **OpenAI-Compatible URL** | Text | *(empty)* | Base URL for OpenAI-compatible providers. |
| **OpenAI-Compatible Model** | Text | *(empty)* | Model identifier sent in requests. |
| **OpenAI-Compatible API Key** | Text | *(empty)* | Optional bearer token. |
| **OpenAI-Compatible Extra Headers** | Multiline | *(empty)* | Extra headers, one per line (`Header: value`). |
| **OpenAI-Compatible Timeout** | Number | `120` | Timeout in seconds (range: 30-3600). |
| **Auto-Restart** | Toggle | On | Automatically restart crashed CLI backends. |
| **Agent Profile** | Dropdown | `pentester` | Active profile from `~/.burp-ai-agent/AGENTS/*.md`. |

## Privacy & Logging

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Privacy Mode** | Dropdown | `BALANCED` | Redaction policy (`STRICT`, `BALANCED`, `OFF`). Users who previously chose `OFF` explicitly keep their choice on upgrade. |
| **Determinism Mode** | Toggle | Off | Stabilizes ordering for reproducible prompt bundles. |
| **Host Anonymization Salt** | Text | *(auto-generated)* | Secret used to produce stable host pseudonyms in STRICT mode. |
| **Audit Logging** | Toggle | Off | Enables JSONL audit log at `~/.burp-ai-agent/audit.jsonl`. |
| **AI Request Logger** | Toggle | On | Enables real-time AI activity logging in the AI Logger tab. |
| **AI Logger Max Entries** | Number | `500` | Maximum entries in the in-memory logger buffer (range: 10+). |

## MCP Server

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Enable MCP** | Toggle | On | Start MCP SSE server when extension loads. |
| **Host** | Text | `127.0.0.1` | Bind address. |
| **Port** | Number | `9876` | TCP port for MCP server. |
| **External Access** | Toggle | Off | Allow non-loopback connections. Requires TLS. |
| **STDIO Bridge** | Toggle | Off | Enable stdio transport in addition to SSE. |
| **Token** | Text | *(auto-generated)* | Bearer token for external access mode. |
| **TLS Enabled** | Toggle | Off | Enable HTTPS for MCP server. |
| **Auto-Generate Certificate** | Toggle | On | Generate self-signed PKCS12 keystore automatically. |
| **Keystore Path** | Text | *(auto-populated)* | Custom PKCS12 path for TLS mode. |
| **Keystore Password** | Text | *(auto-generated)* | Password used by custom keystore. |
| **Max Concurrent Requests** | Number | `4` | Max parallel MCP tool calls (range: 1-64). |
| **Max Body Bytes** | Number | `2097152` | Max body bytes returned by MCP tools. |
| **Enable Unsafe Tools** | Toggle | Off | Master switch for tools marked unsafe. |

## Burp Integration

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Tool Toggles** | Checkboxes | *(varies)* | Enable/disable MCP tools per category. |

## Passive AI Scanner

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Enabled** | Toggle | Off | Enable background passive analysis. |
| **Rate Limit** | Number | `5` | Minimum seconds between analysis requests (range: 1-60). |
| **Scope Only** | Toggle | On | Analyze only targets in Burp scope. |
| **Max Size (KB)** | Number | `96` | Maximum response size eligible for passive analysis (range: 16-1024). |
| **Min Severity** | Dropdown | `LOW` | Ignore findings below selected severity. |
| **Endpoint dedup (min)** | Number | `30` | Skip repeated equivalent endpoint analysis inside window (range: 1-240). |
| **Response dedup (min)** | Number | `30` | Skip repeated response-fingerprint analysis inside window (range: 1-240). |
| **Prompt cache TTL (min)** | Number | `30` | Reuse parsed AI results for identical prompts inside window (range: 1-240). |
| **Prompt cache entries** | Number | `500` | Maximum prompt-result cache entries (range: 50-5000). |
| **Endpoint cache entries** | Number | `5000` | Maximum endpoint dedup cache entries (range: 100-50000). |
| **Fingerprint cache entries** | Number | `5000` | Maximum response-fingerprint dedup cache entries (range: 100-50000). |
| **Req body chars (AI)** | Number | `2000` | Maximum request body chars included in passive AI metadata (range: 256-20000). |
| **Resp body chars (AI)** | Number | `4000` | Maximum response body chars included in passive AI metadata (range: 512-40000). |
| **Max headers** | Number | `40` | Maximum filtered headers included in passive AI metadata (range: 5-120). |
| **Max params** | Number | `15` | Maximum parameters included in passive AI metadata (range: 5-100). |
| **Req body chars (manual)** | Number | `4000` | Maximum request body chars included by manual context actions (range: 256-40000). |
| **Resp body chars (manual)** | Number | `8000` | Maximum response body chars included by manual context actions (range: 512-80000). |
| **Manual context JSON** | Toggle | On (compact) | Use compact JSON serialization for context-menu actions. |
| **Excluded extensions** | Text | `css,js,jpg,jpeg,png,gif,svg,ico,woff,woff2,ttf,eot,otf,mp4,mp3,avi,mov,webm,webp,pdf,zip,gz,tar,rar,7z,map,bmp,tif,tiff` | Comma-separated file extensions to skip in passive scanning. Reduces API calls by skipping static assets. |
| **Batch size (1=off)** | Number | `3` | Group N requests per AI call for batch analysis (range: 1-5). Set to 1 to disable. |
| **Persistent cache** | Toggle | On | Cache AI results to disk (`~/.burp-ai-agent/cache/`) for reuse across Burp sessions. |
| **Persistent TTL (hrs)** | Number | `24` | Hours before persistent disk cache entries expire (range: 1-168). |
| **Persistent max (MB)** | Number | `50` | Maximum disk space for persistent cache in MB (range: 10-500). LRU eviction at 80% capacity. |

## Active AI Scanner

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Enabled** | Toggle | Off | Enable active vulnerability testing. |
| **Max Concurrent Scans** | Number | `3` | Parallel active scans (range: 1-10). |
| **Max Payloads per Point** | Number | `10` | Payload variants per injection point (range: 1-50). |
| **Timeout** | Number | `30` | Timeout in seconds per scan request (range: 5-120). |
| **Request Delay** | Number | `100` | Delay between requests in ms (range: 0-5000). |
| **Max Risk Level** | Dropdown | `SAFE` | Maximum payload risk (`SAFE`, `MODERATE`, `DANGEROUS`). |
| **Scope Only** | Toggle | On | Scan only in-scope targets. |
| **Scan Mode** | Dropdown | `FULL` | Vulnerability-class strategy (`BUG_BOUNTY`, `PENTEST`, `FULL`). |
| **Auto-Queue from Passive** | Toggle | On | Queue high-confidence passive findings into active scanner. |
| **Use Collaborator (OAST)** | Toggle | Off | Enable out-of-band tests using Burp Collaborator. |
| **AI Adaptive Payloads** | Toggle | Off | Generate context-aware payloads using AI based on detected tech stack and error patterns from the Knowledge Base. |

## Prompt Templates

### Built-In Templates

| Setting | Type | Description |
| :--- | :--- | :--- |
| **Find Vulnerabilities** | Text area | Prompt for `Find vulnerabilities` request action. |
| **Analyze this request** | Text area | Prompt for compact endpoint summary. |
| **Explain JS** | Text area | Prompt for JavaScript analysis. |
| **Access Control** | Text area | Prompt for authorization test planning. |
| **Login Sequence** | Text area | Prompt for login flow extraction. |
| **Analyze this Issue** | Text area | Prompt for scanner issue analysis. |
| **Generate PoC & Validate** | Text area | Prompt for proof-of-concept generation. |
| **Impact & Severity** | Text area | Prompt for impact and severity assessment. |
| **Full Report** | Text area | Prompt for complete report generation. |

### Custom Prompt Library

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Custom prompt library** | JSON list (edited via Add/Edit/Dup/Del/Move buttons) | `[]` | Ordered list of saved free-form prompts surfaced in the right-click **Custom prompts** submenu. Each entry stores `id` (UUID), `title`, `promptText`, `tags` (`HTTP_SELECTION` and/or `SCANNER_ISSUE`), and `showInContextMenu`. Persisted as JSON under the preferences key `custom.prompt.library.v1` (settings schema v3). Malformed JSON loads as an empty library. See [Custom Prompt Library](../user-guide/context-menus.md#custom-prompt-library). |

### BountyPrompt Integration

| Setting | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| **Enable BountyPrompt actions** | Toggle | Off | Enables curated BountyPrompt submenu actions in request/response context menus. |
| **Prompt directory** | Text | `~/Tools/BountyPrompt/prompts` | Directory containing BountyPrompt JSON files. |
| **Auto-create issues** | Toggle | On | Auto-create Burp issues for parsed findings from `outputType = issue` prompts. |
| **Issue confidence threshold** | Number | `90` | Minimum parsed confidence (0-100) required for issue creation. |
| **Enabled prompt IDs** | Multi-line text | Curated IDs | Allowlist of prompt IDs, comma- or newline-separated. |

Default curated IDs:

* `API_Keys_Exposure_Detection`
* `CSRF_Vulnerability_Assessment`
* `Security_Headers_Analysis`
* `Vulnerable_Software_Detection`
* `Extract_Endpoints`
* `Vulnerable_File_Upload_Endpoint_Detection`
* `Web_Attack_Suggestions`
* `Sensitive_Error_Messages_Detection`

## File Locations

The extension stores runtime files under `~/.burp-ai-agent/`.

| File/Directory | Purpose |
| :--- | :--- |
| `~/.burp-ai-agent/audit.jsonl` | JSONL audit entries with SHA-256 chaining. |
| `~/.burp-ai-agent/bundles/` | Prompt bundle snapshots for deterministic workflows. |
| `~/.burp-ai-agent/contexts/` | Context snapshot JSON files indexed by hash. |
| `~/.burp-ai-agent/certs/` | Auto-generated TLS artifacts for MCP server. |
| `~/.burp-ai-agent/cache/` | Persistent prompt cache (JSON files keyed by prompt hash). Per-project namespaced. |
| Burp `extensionData()` | Project-scoped chat session storage (auto-migrated from global `preferences()`). |
| `~/.burp-ai-agent/backends/` | Drop-in backend JAR discovery directory. |
| `~/.burp-ai-agent/AGENTS/` | Agent profile markdown files. |
| `~/.burp-ai-agent/AGENTS/default` | Active profile marker file. |
| `~/.burp-ai-agent/logs/` | Rolling AI Request Logger JSONL files (opt-in). |

## Runtime Constants

Some limits are operational constants rather than direct UI controls:

* **Auto tool chain limit**: `8` sequential MCP tool calls per chat interaction.
* **Active scanner queue max**: `2000` targets.
* **Active dedup window**: `1 hour`.
* **Passive analysis wait timeout**: `90s`.
* **HTTP backend conversation history cap**: `20` messages and `40000` total characters (minimum 2 latest messages retained).
* **CLI history cap**: `10` messages or `20000` characters.
* **Large prompt threshold for Claude CLI / Copilot CLI fallback path**: `32000` characters.
* **Backend retry attempts**: `6` with exponential backoff (HTTP backends).
* **OpenCode CLI idle timeout**: `30s` before process termination after last output.
* **`CHAT_MAX_OUTPUT_TOKENS`**: `4096` — max output tokens for chat responses.
* **`SCANNER_MAX_OUTPUT_TOKENS`**: `2048` — max output tokens for single scanner analysis.
* **`SCANNER_BATCH_MAX_OUTPUT_TOKENS`**: `4096` — max output tokens for batch scanner analysis.
* **`PAYLOAD_MAX_OUTPUT_TOKENS`**: `1024` — max output tokens for adaptive payload generation.

## Rolling Log Persistence (JVM Properties)

The AI Request Logger can persist entries to rotating JSONL files. This is configured via JVM system properties at Burp startup:

| Property | Default | Description |
| :--- | :--- | :--- |
| `burp.ai.logger.rolling.enabled` | `false` | Enable rolling file persistence. |
| `burp.ai.logger.rolling.dir` | `~/.burp-ai-agent/logs` | Directory for log files. |
| `burp.ai.logger.rolling.maxBytes` | `1048576` (1 MB) | Maximum size per log file (minimum 10 KB). |
| `burp.ai.logger.rolling.maxFiles` | `5` | Maximum number of rolled files (range: 1–20). |

Example:

```bash
java -jar burpsuite.jar \
  -Dburp.ai.logger.rolling.enabled=true \
  -Dburp.ai.logger.rolling.maxBytes=2097152 \
  -Dburp.ai.logger.rolling.maxFiles=10
```
