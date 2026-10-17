# Burp AI (Built-in)

The **Burp AI** backend is an in-process backend that runs through Burp's built-in AI (the Montoya `api.ai()` interface). It requires no external URL, API key, or CLI and produces the lowest-latency response path among all backends because no HTTP or child process boundary is crossed.

{% hint style="info" %}
The BApp Store build runs on Burp Community and Burp Professional; it sends AI requests only while Burp's "Use AI" setting is enabled for the extension, and with it off it loads and refuses every AI request.

See [Which Build?](../getting-started/which-build.md#burp-edition-requirements) for what each build does with "Use AI" on and off.
{% endhint %}

## Prerequisites

1. AI credits available to Burp AI on your Burp account.
2. Burp's **"Use AI"** setting enabled for the extension. PortSwigger documents "Use AI" as a per-extension checkbox, off by default: **Extensions > Installed > Custom AI Agent > Use AI**.

The extension reads `api.ai().isEnabled()` at every AI request, so switching "Use AI" on or off takes effect at the next request without restarting Burp. If that call throws, the extension reads the setting as off.

## The "Use AI" Gate in Each Build

"Use AI" is needed by the **Burp AI** backend in both builds. What else it gates depends on the build you loaded:

| "Use AI" off | BApp Store build | GitHub full build |
| :--- | :--- | :--- |
| Burp AI | Refused | Refused |
| Every other HTTP backend (Anthropic, Ollama, LM Studio, NVIDIA NIM, Perplexity, Generic OpenAI-compatible) | Refused | Keeps working |
| CLI backends (Gemini, Claude, Codex, OpenCode, Copilot) | Not in this build | Keep working |
| MCP `ai_analyze` and `ai_passive_scan` | Return a tool error | Return a tool error |

While the selected backend is refused, the status pill reads **"AI disabled in Burp"** (its tooltip carries the refusal text), the chat shows the refusal inline, and an AI scanner that is switched on shows "Paused: Use AI is off" in its status line. In the BApp Store build that happens whatever backend is selected; in the GitHub full build only when Burp AI is selected. The pill re-reads the setting locally about once a second, without sending a request, so the change shows up without a click.

The refusal texts are:

* BApp Store build: "This build sends AI requests only while Burp's 'Use AI' setting is enabled. Turn it on in Burp's settings to use any AI backend."
* GitHub full build, Burp AI selected: "Burp AI is disabled in Burp Suite settings. Enable 'Use AI' for extensions, or pick a different backend."

## Selecting Burp AI

1. Open **Custom AI Agent → Settings → AI Backend**.
2. In **Preferred Backend** choose `burp-ai` (Burp AI built-in).
3. Check the status pill in the top bar. In the BApp Store build it reads "Not checked · click to test" until the first check: click it to run one. In the GitHub full build Burp AI counts as a local backend, so the pill checks it at startup and every 30 seconds.

The Burp AI card has no **Test connection** button. Its health check asks Burp itself and sends no network request; if it runs while "Use AI" is off, it reports `Burp AI is disabled. Enable 'Use AI' in Burp Suite settings.` In practice the pill shows "AI disabled in Burp" first, because the gate is read before any check is submitted.

There is no URL, model, token, or custom command — configuration lives entirely inside Burp's own AI settings.

## Capabilities

| Capability | Value |
| :--- | :--- |
| Streaming | No — Burp returns a full response from `api.ai().prompt().execute(...)` in one call. |
| JSON mode | Enforced via the prompt: when `jsonMode=true` the extension appends *"IMPORTANT: Respond ONLY with valid JSON."* to the user message. |
| System role | Yes. Agent profiles are delivered as `Message.systemMessage(...)` and precede the conversation history. |
| Auto-start | Not applicable (no process to launch). |
| Temperature | `0.0` when **Determinism Mode** is on, otherwise `0.3`. |

## Privacy Posture

Burp AI keeps requests inside Burp's own AI route. The plugin does not open an additional backend connection when this backend is selected—the prompt, context, and conversation history flow through the Montoya `api.ai()` channel. The same privacy-mode pipeline used by other backends runs before handoff, including producer-specific sanitizers and the shared text pass. Its [carrier boundaries](../privacy/limitations.md#redaction-coverage-and-known-boundaries) still apply; selecting Burp AI does not turn pattern-based redaction into a no-secret-egress guarantee.

See [Privacy Modes](../privacy/privacy-modes.md) for what gets redacted.

## Limitations

* Needs Burp's "Use AI" setting enabled for the extension, in both builds.
* Streaming UI indicators in the chat panel appear as a single chunk because the backend returns the full response at once.
* There is no per-backend timeout setting; request timing is governed by Burp's own AI limits.
* The extension's HTTP retry/circuit breaker does not wrap this backend — retries fall back to whatever Burp does internally.
* Tool-chain execution still runs through the MCP catalog like any other backend, but the response text is produced in a single call rather than streamed.

## Troubleshooting

* **The pill reads "AI disabled in Burp"** — turn on "Use AI" for the extension (**Extensions > Installed > Custom AI Agent > Use AI**, as documented by PortSwigger). The pill returns to its idle text within about a second. The BApp Store build then checks on demand (Save, a backend change, a pill click, or the first chat message); the GitHub full build re-checks Burp AI every 30 seconds.
* **AI credits exhausted** — Burp surfaces quota errors directly; the plugin relays them as an `ERROR` entry in the [AI Request Logger](../privacy/ai-request-logger.md).
* **`api.ai().isEnabled()` throws on custom builds** — on older Montoya API versions the `ai()` surface may be missing. The extension then reads "Use AI" as off, so Burp AI is refused (in the BApp Store build, every backend is). Registered backends remain listed in **Preferred Backend**.

## Related Pages

* [Which Build?](../getting-started/which-build.md)
* [First Run Checklist](../getting-started/first-run-checklist.md)
* [Backends Overview](overview.md)
* [Burp Integration](../user-guide/burp-integration.md)
* [Troubleshooting](../reference/troubleshooting.md)
