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

### Empty or low-value responses

* Reduce selected context size.
* Confirm selected backend is actually running.
* For local HTTP backends, verify service and model availability.

### Backend will not start

* Verify auto-restart and command settings.
* Check port conflicts for HTTP backends.
* Ensure required API keys exist in Burp runtime environment.

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

### Settings not persisting

* Confirm Burp project/preferences are writable.
* Restart extension if a setting requires lifecycle restart.

### High memory usage

* Reduce scanner max size.
* Disable passive scanner when idle.
* Close unused chat sessions.
