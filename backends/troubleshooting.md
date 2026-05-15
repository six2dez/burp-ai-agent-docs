# Backend Troubleshooting

When the **AI: OK / Degraded / Offline** pill in the top bar reports a problem, work through this page from top to bottom. The first three sections (Health States, Diagnostic Flow, Switching Backend) apply to every backend. The Per-Backend Error Signatures section covers the specific failure shapes you are most likely to hit.

## Health States Recap

The plugin polls the active backend every five seconds. The polled state is rendered as a colored pill plus tooltip:

| Pill | Internal state | What you should do |
| :--- | :--- | :--- |
| **AI: OK** | `Healthy` | Nothing — backend is responding. |
| **AI: Degraded** | `Degraded` | Read the tooltip. Usually transient (slow first token, soft rate-limit) — retry the in-flight request before changing anything. |
| **AI: Offline** | `Offline` / `Unavailable` | Read the tooltip. This means the next request will either fail fast (HTTP) or never start (CLI). Fix or switch backend. |

Each state transition lands as an entry in the [AI Request Logger](../privacy/ai-request-logger.md). Use the **Type** filter set to `ERROR` / `RETRY` to see the recent failure timeline alongside backend metadata.

<!-- TODO: screenshot strip comparing the three top-bar pill states (AI: OK in green, AI: Degraded in amber, AI: Offline in red) ideally with their tooltips visible. Save as .gitbook/assets/backend-health-states.png and replace this marker with:
![Screenshot: Backend health pill states](../.gitbook/assets/backend-health-states.png)
-->


## Diagnostic Flow

1. **Read the tooltip.** The pill tooltip carries the underlying error message from the health probe. That message is almost always enough to identify the root cause.
2. **Look at the AI Request Logger.** Filter by trace ID for the most recent prompt. The `ERROR` entry usually carries the upstream status code or process exit code.
3. **Trigger a one-off chat prompt.** A single "hello" is the cheapest reproduction. If the chat fails with the same error, the issue is the backend itself, not a particular scanner pipeline.
4. **Check the backend-specific section below.**
5. **Switch backends.** If the failure is upstream (API outage, rate-limit) and you have a configured alternative, change **Preferred Backend** under **Settings → AI Backend** rather than waiting it out.

## When the Circuit Breaker Has Tripped

HTTP backends are wrapped in a circuit breaker that opens after **5 consecutive failures**. While open, the next probe fails fast with `<backend> backend is temporarily unavailable (circuit open)` and the AI Request Logger shows synthetic errors instead of new requests.

* The breaker stays open for **30 seconds**, then allows a single half-open probe. Success closes it; failure reopens it.
* The breaker resets immediately when you switch backends — the circuit is per-backend.
* If you genuinely fixed the upstream issue and do not want to wait, switch backend and switch back to force a fresh probe.

## Per-Backend Error Signatures

### HTTP backends (Ollama, LM Studio, NVIDIA NIM, Perplexity, Generic OpenAI-compatible)

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| `Connection refused` on probe | Local server not running (Ollama, LM Studio) or wrong URL | Start the server (`ollama serve`, `lms server start`) or verify the **URL** field. |
| `401 Unauthorized` | API key missing, wrong, or expired | Re-paste the **API Key**; check it starts with the expected prefix (`pplx-…`, `nvapi-…`, `sk-…`). |
| `404 Not Found` on the chat endpoint | Wrong base URL or wrong path. Common with Perplexity if pointed via Generic OpenAI-compatible (which expects `/v1/chat/completions`). | Use the dedicated **Perplexity** backend, not Generic. Verify the URL does not double-up `/v1/`. |
| `400 Bad Request` mentioning `response_format` | Backend does not support JSON mode but a request forced it | Use a backend that supports JSON mode for scanner workflows, or accept text-mode parsing on Perplexity. |
| `model_not_found` / `invalid_model` | Model identifier typo | Check the model name matches the provider's catalog exactly (case-sensitive). |
| Slow first token, then `Degraded` recovers to `OK` | Cold-start latency on shared infra (NVIDIA NIM, Perplexity) | Normal. Repeat the request; subsequent ones are fast. |
| Persistent `Degraded` with retries in the logger | Provider rate limiting (soft 429s wrapped as retryable) | The plugin retries 6 times with stepped backoff. If retries do not clear it, switch to a different model or backend. |
| `<backend> backend is temporarily unavailable (circuit open)` | 5 consecutive failures tripped the breaker | See "When the Circuit Breaker Has Tripped" above. |

### Burp AI (built-in, Burp Pro only)

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| `AI: Offline` with tooltip `Burp AI is not enabled` | **Settings → Burp AI → Use AI for extensions** is off | Toggle it on inside Burp Suite. The plugin picks it up on the next health cycle (within 5 s) — no restart. |
| `AI: Offline` and Burp Community | Backend is Pro-only | Switch to any non-Burp-AI backend. |
| Quota errors surfaced as `ERROR` entries | Burp Pro AI credits exhausted | Top up credits via PortSwigger, or switch backend. |
| Backend missing from the **Preferred Backend** dropdown | Older Montoya API without `ai()` surface | The plugin auto-hides Burp AI when the API surface is absent. Upgrade Burp Suite. |

### CLI backends (Gemini, Claude, Codex, Copilot, OpenCode)

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| `command not found` in health output | CLI not on the `PATH` inherited by Burp | Pass an absolute path in the corresponding **CLI Command** setting, or relaunch Burp from a shell where the CLI works. |
| `CreateProcess error=193` (Windows) | npm-installed CLI shim isn't directly executable from Java | Point to the `.cmd` shim (e.g., `C:\\Users\\<you>\\AppData\\Roaming\\npm\\claude.cmd`). See [Backends Overview → Windows npm Shim Resolution](overview.md#windows-npm-shim-resolution). |
| Process exits with `1` on first prompt | CLI not authenticated in this user account | Run the CLI's auth command in the same shell environment Burp inherits (`claude /login`, `gemini auth login`, etc.), then retry. |
| Output is blank or truncated | OpenCode idle timeout fires before first token | The plugin terminates an OpenCode subprocess after **30 seconds** of idle stdout. Retry after warming the model, or pick a different backend for cold-start workloads. |
| Very long prompts produce no output | Claude/Copilot CLI fallback path threshold (`32000` chars) | The CLI is being fed a smaller prompt by design. Trim context (lower the **Manual context body chars** caps) or switch to an HTTP backend that streams. |
| `AI: Offline` after a successful run | Supervisor decided the process exited unexpectedly | **Auto-Restart** is on by default — wait one health cycle. If it never recovers, check Burp's Output tab for the supervisor's diagnostic. |

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
  * [NVIDIA NIM](nvidia-nim.md), [Perplexity](perplexity.md), [Generic OpenAI-compatible](openai-compatible.md)
  * [Gemini CLI](gemini-cli.md), [Claude CLI](claude-cli.md), [Codex CLI](codex-cli.md), [Copilot CLI](copilot-cli.md), [OpenCode CLI](opencode-cli.md)
* [Troubleshooting (general)](../reference/troubleshooting.md)
* [AI Request Logger](../privacy/ai-request-logger.md)
