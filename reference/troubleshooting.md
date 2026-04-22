# Troubleshooting

Common issues and practical resolution steps.

## Installation Issues

### Extension fails to load

* Ensure Burp runs on Java 21.
* Re-download JAR if archive is corrupted.
* Temporarily disable other extensions to rule out conflicts.

### Tab not appearing

* Verify extension is enabled in **Extensions -> Installed**.
* Check Burp **Errors** tab for initialization failures.
* Check Burp **Output** tab for startup logs.

## Backend Issues

### Backend status is `Crashed`

* Verify CLI command is valid in a normal terminal.
* If Burp is started from GUI, set full executable paths.
* Confirm provider authentication is already completed.
* Verify model identifiers are correct.
* Check extension output for exit code/error text.
* Use backend **Test connection** in the **AI Backend** settings tab.

### Backend status is `AI: Degraded` or `AI: Offline`

* Run **Test connection** in backend settings.
* Check local service reachability for HTTP backends.
* For CLI backends, test the exact configured command in terminal.
* Review extension output for health check diagnostics.

### AI responses are truncated

All HTTP backends now set `max_tokens` (OpenAI-compatible, LM Studio) or `num_predict` (Ollama) automatically per request type: 4096 for chat, 2048 for single scanner analysis, 4096 for batch, and 1024 for adaptive payloads. If responses were previously cut short, this is now handled without manual configuration.

### Empty or low-value responses

* Reduce selected context size.
* Confirm selected backend is actually running.
* For local HTTP backends, verify service and model availability.

### OpenCode CLI: blank or missing responses

* The extension filters OpenCode status lines (`> thinking`, `> loading`, etc.) and prompt echoes from the output. If the actual AI response is very short or consists entirely of lines that match filtered patterns, the result may appear blank.
* The idle timeout is 30 seconds. If the model takes longer than 30 seconds without producing any output after initial status lines, the process is terminated. For very large prompts, consider using a faster model.
* Test the exact command in a terminal to confirm the model produces output: `opencode run "your prompt"`.
* Check the Burp extension **Output** tab for raw process output and error messages.

### Backend will not start

* Verify auto-restart and command settings.
* Check port conflicts for HTTP backends.
* Ensure required API keys exist in Burp runtime environment.

### Scanner uses wrong backend after picker change

If the scanner logs show a different backend than what you selected, re-select the backend in the top bar picker. The picker now auto-saves on change. Older sessions from before this fix may need a manual re-selection.

### Windows CLI: CreateProcess error=193

npm-installed CLI tools (Codex, Gemini, OpenCode, Copilot) install as shell script shims on Windows that Java cannot execute directly. The extension resolves `.cmd` shim siblings automatically. If auto-resolution fails:

* Use the full `.cmd` path explicitly: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\codex.cmd`.
* Verify the `.cmd` file exists alongside the shell script shim.

### Windows + WSL Codex CLI bridge

If Burp runs on Windows and Codex runs in WSL, use a `.cmd` wrapper that forwards args into WSL and configure **Codex CLI Command** with that wrapper path.

## MCP Issues

### MCP toggle will not stay ON

* Check MCP port conflicts.
* Validate bind host configuration.
* Restart Burp if an old crashed process still holds the port.

### MCP client cannot connect

* Validate token usage for external access mode.
* Ensure MCP URL and port match extension settings.
* Confirm TLS setup if HTTPS is enabled.

### MCP tools missing

* Enable tool toggles in **Burp Integration**.
* Enable unsafe tools if required.
* Verify Burp edition supports requested tools.

## Scanner Issues

### No issues are created from passive scanner

* Confidence threshold for auto-issue is `>= 85%`.
* Ensure targets are in scope when scope-only is enabled.
* Verify min severity filter is not too strict.
* Confirm backend is available.

### Passive scanner token usage is too high

* Increase **Rate Limit** and keep **Scope Only** enabled.
* Lower **Resp body chars (AI)** and **Req body chars (AI)**.
* Reduce **Max headers** and **Max params** if prompts are still noisy.
* Keep dedup/cache controls enabled (`Endpoint dedup`, `Response dedup`, `Prompt cache TTL`).
* Lower **Max Size (KB)** when scanning endpoints with large responses.

### Scanner reports findings from other domains

Fixed by clearing the ScanKnowledgeBase when the passive scanner is disabled and namespacing the PersistentPromptCache per project. Findings from a previous project or domain no longer leak into the current session.

### Active scanner does not run

* Enable Active toggle.
* Queue targets from context menu.
* Check dedup window behavior.
* Check scope-only filter.

### Active scan queued `0` targets

* Target may be out of scope.
* Request may be invalid for extraction.
* Queue may be at hard cap (`2000`).

## BountyPrompt Issues

### BountyPrompt submenu is disabled

* Enable **Enable BountyPrompt actions** in **Prompt Templates**.
* Confirm MCP server is running.

### No BountyPrompt items appear

* Verify **Prompt directory** exists and contains JSON files.
* Verify selected IDs are present in **Enabled prompt IDs**.
* Only curated IDs are loaded; non-curated JSON files are ignored.

### Prompt exists on disk but is not listed

* Check filename ID matches a curated ID exactly.
* Validate JSON includes non-empty `systemPrompt` and `userPrompt`.
* Review Burp extension output for `[BountyPrompt]` loader errors.

### BountyPrompt output does not create issues

* Prompt may use `outputType = prompt output` (chat-only by design).
* **Auto-create issues** may be disabled.
* Parsed confidence may be below **Issue confidence threshold**.
* Output may contain `NONE`, which is treated as no finding.
* Duplicate issue (same base URL + name) may be skipped.

### BountyPrompt context looks incomplete

* Prompt tags control which fields are sent.
* Missing response data yields `<no response>` for response tags.
* Privacy mode redaction may hide tokens/cookies/hosts by design.

## General Issues

### Extension output appears empty

* Launch Burp from terminal to see stdout/stderr.
* Use Burp extension **Output/Errors** tabs.

### Chat sessions from another project appear

Chat sessions are now stored in `extensionData()` which is project-scoped. Sessions from one Burp project do not appear in another. On first open, existing sessions are auto-migrated from global `preferences()` to the current project. Switch back to the original project to see its sessions.

### Webhook notifications not arriving

`sendWebhook()` is now wrapped in a try-catch for best-effort delivery. Check the Burp extension **Output** tab for webhook error messages. Verify the webhook URL is reachable and correctly configured.

### Settings not persisting

* Confirm Burp project/preferences are writable.
* Restart extension if a setting requires lifecycle restart.

### High memory usage

* Reduce scanner max size.
* Reduce passive dedup/cache entry sizes if running with very large traffic volumes.
* Disable passive scanner when idle.
* Close unused chat sessions.
* Reduce **AI Logger Max Entries** if the in-memory log is consuming too many resources.

## AI Request Logger Issues

### AI Logger tab is empty

* Verify **AI Request Logger** is enabled in **Privacy & Logging**.
* Confirm a backend is configured and chat or scanner operations have been performed.
* Check that the type/source/preset filters are not hiding entries.

### Rolling log files are not created

* Verify JVM property `-Dburp.ai.logger.rolling.enabled=true` is set at Burp startup.
* Check the log directory is writable (`~/.burp-ai-agent/logs/` by default).
* Ensure the directory exists or can be created by the extension.

### Tool chain stops before completing

* The auto tool chain limit is 8 iterations. If the AI needs more steps, the chain stops and the AI produces a response with whatever information it has gathered.
* Check the AI Logger for error entries in the chain trace — a tool failure terminates the chain early.
* Verify MCP tools are enabled and the MCP server is running.

### Trace ID filter shows no results

* Trace IDs are case-insensitive substring matches. Verify the ID is correct.
* Older entries may have been evicted from the in-memory buffer. Use rolling JSONL persistence for long-term retention.
