# Backend Troubleshooting

When the **AI: OK / Degraded / Offline / AI disabled in Burp** pill in the top bar reports a problem, work through this page from top to bottom. The first three sections (Health States, Diagnostic Flow, Switching Backend) apply to every backend. The Per-Backend Error Signatures section covers the specific failure shapes you are most likely to hit.

## Health States Recap

The backend-specific result is rendered as a colored pill plus tooltip. When the checks run depends on the build: the BApp Store build checks on demand only (Save, a backend change, a pill click, or a chat reply); the GitHub full build checks every backend at startup and then local backends every 30 seconds, and remote providers again after a settings or backend change and on a pill click. See [Backends Overview → Health States](overview.md#health-states). Before any check, the pill reads Burp's "Use AI" setting locally, about once a second, and sends nothing for a refused backend.

| Pill | Internal state | What you should do |
| :--- | :--- | :--- |
| **Not checked · click to test** / **AI: ?** | none yet | No check has run yet (BApp Store build / GitHub full build). Click the pill to run one. |
| **AI: OK** | `Healthy` | The configured check passed. For CLI backends this checks executable resolution, not authentication; for Anthropic it checks only that a key is present. |
| **AI: Degraded** | `Degraded` | Read the tooltip. The HTTP health `GET` reached the endpoint but got HTTP 401, 403 (authentication) or 429 (rate limit). |
| **AI: Offline** | `Unavailable` | Read the tooltip. The endpoint/configuration check failed or the CLI executable is unresolved. |
| **AI disabled in Burp** | refused by the gate | Burp's "Use AI" is off and this build refuses the selected backend: any backend in the BApp Store build, Burp AI in the GitHub full build. Turn on "Use AI" for the extension. See [Which Build?](../getting-started/which-build.md). |

Status transitions are not logger events. Use the [AI Request Logger](../privacy/ai-request-logger.md) **ERROR** / **RETRY** filters for failures and retries produced by actual model requests.


## Diagnostic Flow

1. **Read the tooltip.** The pill tooltip carries the underlying error message from the health probe. That message is almost always enough to identify the root cause.
2. **Look at the AI Request Logger.** Filter by trace ID for the most recent prompt. The `ERROR` entry usually carries the upstream status code or process exit code.
3. **Trigger a one-off chat prompt.** A single "hello" is the cheapest reproduction. If the chat fails with the same error, the issue is the backend itself, not a particular scanner pipeline.
4. **Check the backend-specific section below.**
5. **Switch backends.** If the failure is upstream (API outage, rate-limit) and you have a configured alternative, change **Preferred Backend** under **Settings → AI Backend** rather than waiting it out.

## When the Circuit Breaker Has Tripped

Each launched HTTP connection has a circuit breaker that opens after **5 recorded transient failures**. While open, the next model request on that connection fails fast with `<backend> backend is temporarily unavailable (circuit open)` and the AI Request Logger records the error instead of sending it upstream.

* The breaker stays open for **30 seconds**, then allows one half-open model request. Success closes it; failure reopens it.
* Transport exceptions count per failed attempt. HTTP 429 and 5xx responses count once; other 4xx responses do not open the breaker.
* The breaker belongs to the launched connection, not the top-bar health checker. Start a new chat session for a fresh connection, or wait for the half-open attempt; changing the preferred backend does not rewrite an existing session's breaker.

## Per-Backend Error Signatures

### HTTP backends (Ollama, LM Studio, NVIDIA NIM, Perplexity, Anthropic, Generic OpenAI-compatible)

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| `Connection refused` on probe | Local server not running (Ollama, LM Studio) or wrong URL | Start the server (`ollama serve`, `lms server start`) or verify the **URL** field. |
| `401 Unauthorized` | API key missing, wrong, or expired | Re-paste the **API Key**; check it starts with the expected prefix (`pplx-…`, `nvapi-…`, `sk-…`). |
| `404 Not Found` on the chat endpoint | Wrong base URL or wrong path. Common with Perplexity if pointed via Generic OpenAI-compatible (which expects `/v1/chat/completions`). | Use the dedicated **Perplexity** backend, not Generic. Verify the URL does not double-up `/v1/`. |
| `400 Bad Request` mentioning `response_format` | Backend does not support JSON mode but a request forced it | Use a backend that supports JSON mode for scanner workflows, or accept text-mode parsing on Perplexity. |
| `model_not_found` / `invalid_model` | Model identifier typo | Check the model name matches the provider's catalog exactly (case-sensitive). |
| `Degraded` with HTTP 401/403 in the tooltip | The live health endpoint was reachable but rejected authentication | Re-paste the key and verify it belongs to the configured endpoint. |
| `Degraded` with HTTP 429 in the tooltip | The provider rate-limited the health `GET` (for NVIDIA NIM and Perplexity, a free `GET /v1/models`) | Review quota and provider logs. Remote providers are not checked on a timer in either build, so the next check runs only when you ask for one. HTTP 429 is not retried inside the same model call. |
| `<backend> backend is temporarily unavailable (circuit open)` | 5 consecutive failures tripped the breaker | See "When the Circuit Breaker Has Tripped" above. |

### Burp AI (built-in)

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| `AI disabled in Burp` | Burp's "Use AI" is off for the extension. In the BApp Store build this refuses every backend, not only Burp AI | Turn it on: PortSwigger documents "Use AI" as a per-extension checkbox, off by default: **Extensions > Installed > Custom AI Agent > Use AI**. No restart is needed: the pill re-reads the setting within about a second, and the next request uses it. The BApp Store build then checks the backend on demand (Save, a backend change, a pill click, or a chat reply); the GitHub full build re-checks Burp AI every 30 seconds. |
| `AI disabled in Burp` and "Use AI" cannot be turned on | Which Burp editions can turn "Use AI" on is decided by Burp, not by this extension | In the GitHub full build, select any backend other than Burp AI: they do not depend on "Use AI". In the BApp Store build every AI backend needs it. See [Which Build?](../getting-started/which-build.md#burp-edition-requirements). |
| Quota errors surfaced as `ERROR` entries | Burp AI credits exhausted | Top up credits via PortSwigger, or switch backend. |
| Burp AI is present in **Preferred Backend** but remains refused | "Use AI" is off, or the Montoya AI surface is missing (the call throws, which the extension reads as off) | Enable "Use AI" for the extension, or update Burp; in the GitHub full build you can also select a backend that does not depend on it. Registered backends remain listed even when they are refused or their health check reports unavailable. |

### CLI backends (Gemini, Claude, Codex, Copilot, OpenCode; GitHub full build only)

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| `command not found` in health output | CLI not on the `PATH` inherited by Burp | Pass an absolute path in the corresponding **CLI Command** setting, or relaunch Burp from a shell where the CLI works. |
| `CreateProcess error=193` (Windows) | npm-installed CLI shim isn't directly executable from Java | Point to the `.cmd` shim (e.g., `C:\\Users\\<you>\\AppData\\Roaming\\npm\\claude.cmd`). See [Backends Overview → Windows npm Shim Resolution](overview.md#windows-npm-shim-resolution). |
| `AI: OK`, but the first prompt exits with an authentication error | CLI health checks executable resolution, not provider authentication | Run the CLI's auth command in the same shell environment Burp inherits (`claude /login`, `gemini auth login`, etc.), then retry. |
| Output is blank or truncated | OpenCode idle timeout fires before first token | The plugin terminates an OpenCode subprocess after **30 seconds** of idle stdout. Retry after warming the model, or pick a different backend for cold-start workloads. |
| Very long prompts produce no output | Claude/Copilot CLI fallback path threshold (`32000` chars) | The full combined prompt was written to an owner-only temp file where POSIX permissions are supported, and the CLI received an instruction containing that path. Check that the CLI can read the local temp path, or trim the manual-context caps. |

## Switching Backend Without Losing Work

The active chat session stores its conversation history per-session, not per-backend. Changing **Preferred Backend** affects only **new** sessions and the scanner pipelines. To move a stuck session to a different backend:

1. Note the title of the stuck session.
2. Right-click the session → **Export as Markdown**.
3. Switch **Preferred Backend**.
4. Open a fresh chat session and paste the relevant context back.

For scanners, switching the backend takes effect on the next analysis cycle. Anything mid-flight finishes (or fails) on the original backend.

## Cross-references

* [Backends Overview](overview.md) for capability matrix, retry schedule, output token limits, and CLI detection rules.
* Per-backend pages — each has its own troubleshooting block for provider-specific quirks:
  * [Burp AI (built-in)](burp-ai.md)
  * [Ollama](ollama.md), [LM Studio](lm-studio.md)
  * [NVIDIA NIM](nvidia-nim.md), [Perplexity](perplexity.md), [Anthropic](anthropic.md), [Generic OpenAI-compatible](openai-compatible.md)
  * [Gemini CLI](gemini-cli.md), [Claude CLI](claude-cli.md), [Codex CLI](codex-cli.md), [Copilot CLI](copilot-cli.md), [OpenCode CLI](opencode-cli.md)
* [Troubleshooting (general)](../reference/troubleshooting.md)
* [AI Request Logger](../privacy/ai-request-logger.md)
