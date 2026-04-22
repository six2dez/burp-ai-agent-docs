# FAQ

## Does it work with Burp Community Edition?

Yes. Core chat, context actions, passive scanner, and most MCP functionality work in Community. Pro-only capabilities (scanner APIs, some tools) remain gated.

## Does the passive scanner send extra traffic?

No. Passive scanner analyzes observed proxy traffic. Active scanner is the component that sends additional requests.

## Are my data safe with cloud backends?

Use `STRICT` or `BALANCED` privacy modes and review context before sending prompts. For maximum control, use local backends (Ollama/LM Studio).

## Which model/backend should I choose?

* Maximum privacy: local backends.
* Large cloud context: Gemini CLI.
* Reasoning-heavy workflows: Claude CLI.
* Multi-model flexibility via GitHub: Copilot CLI.
* General purpose and code-heavy analysis: Codex CLI.

## How can I reduce cloud cost?

Tune passive scanner caps, dedup/cache settings, and context size controls. See [Token Usage & Cost Management](../user-guide/token-management.md).

## Can I use multiple backends at once?

Yes. Different chat sessions can use different backends. The backend selector controls new sessions.

## Why does backend status show Degraded/Offline?

Health checks detected command/API issues. Use backend **Test connection** and inspect extension output logs.

## Why are MCP tools missing?

Check tool toggles in **Burp Integration**, unsafe-tool master switch in **MCP Server**, and Burp edition constraints.

## What is the AI Request Logger?

The AI Request Logger is a real-time activity log that records all AI interactions: prompts, responses, MCP tool calls, retries, errors, and scanner operations. It appears as the **AI Logger** tab in the settings panel and supports filtering by type, source, preset, and trace ID. See [AI Request Logger](../privacy/ai-request-logger.md).

## Why does the AI call tools instead of answering directly?

The extension supports auto tool chaining: when the AI needs data from Burp (e.g., proxy history, site map), it calls MCP tools automatically and feeds the results back into its reasoning. This can chain up to 8 tool calls before the final answer. Check the **AI Logger** tab to see each step in the chain.

## Can I persist AI Logger entries to disk?

Yes. Add JVM properties at Burp startup to enable rolling JSONL persistence. See [Settings Reference](settings-reference.md#rolling-log-persistence-jvm-properties).

## Why does the profile warning about `http1_request` appear?

Agent profiles list available MCP tools including `http1_request` and `http2_request`. These tools require **Unsafe mode** to be enabled. When Unsafe mode is off, the tools are disabled and the profile validation shows a warning. The built-in profiles treat these as optional — the warning is suppressed for catalog-only references. If you see it for explicit tool references, enable **Unsafe Tools** in the MCP Server tab or remove the reference from your custom profile.

## How do I correlate related log entries?

Every operation generates a trace ID (e.g., `chat-turn-{UUID}`) that is shared across all related entries. Use the **Trace** filter in the AI Logger tab to isolate entries for a specific operation.

## Why does the scanner use a different backend than the one I selected?

The backend picker in the top bar now immediately persists your selection. If you changed the backend before this fix was applied, re-select the backend in the top bar to ensure scanners pick it up.

## Why does Codex/Gemini/OpenCode CLI fail with "CreateProcess error=193" on Windows?

npm-installed CLI tools install as shell script shims on Windows that Java cannot execute directly. The extension automatically resolves `.cmd` sibling files for all npm-installed backends. If auto-resolution fails, set the full `.cmd` path in the backend command field (e.g., `C:\\Users\\<you>\\AppData\\Roaming\\npm\\codex.cmd`).

## Why does OpenCode CLI return blank responses?

The extension filters OpenCode status/metadata lines from the output. Blank responses can occur when:

* The model produces output that is entirely filtered (e.g., only status lines).
* The model takes too long to start producing content (idle timeout is 30 seconds).
* The prompt contains short common terms that match response lines.

Test the exact command in a terminal to verify output. Check the Burp extension **Output** tab for diagnostics.

## Are chat sessions shared across Burp projects?

No. Chat sessions are stored in `extensionData()` which is project-scoped. Each Burp project has its own independent set of chat sessions. On first open, existing sessions are auto-migrated from global `preferences()` to the current project.

## Why did sessions disappear after opening another project?

Sessions are project-scoped. They did not disappear -- they belong to the previous project. Switch back to that project to see them again.

## Does disabling the passive scanner clear knowledge?

Yes. Disabling the passive scanner clears the ScanKnowledgeBase to prevent stale cross-domain findings. The PersistentPromptCache is also namespaced per project, so cached results from one project do not leak into another.

## Why are responses no longer truncated?

All HTTP backends now set `max_tokens` (OpenAI-compatible, LM Studio) or `num_predict` (Ollama) automatically per request type. Chat uses 4096 output tokens, single scanner analysis uses 2048, batch analysis uses 4096, and adaptive payload generation uses 1024. This eliminates premature truncation without manual configuration.
