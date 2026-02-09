# Troubleshooting

Common issues and their resolutions.

## Installation Issues

### Extension fails to load
*   **Java Version**: Ensure you are using **Java 21**. Burp's "Errors" tab in the Extensions window will show `UnsupportedClassVersionError` if your Java is too old.
*   **Corrupt JAR**: If downloading from GitHub, ensure the download completed fully. Compare the SHA-256 hash if provided.
*   **Conflicting extensions**: Disable other extensions temporarily to check for conflicts.

### Tab not appearing
*   Ensure the extension is checked in **Extensions → Installed**.
*   Check the **Errors** tab for any initialization crashes.
*   Check the **Output** tab for error messages during loading.

## Backend Issues

### Backend status is "Crashed"
*   **CLI Path**: Verify the command (e.g., `gemini`, `claude`, `codex`) is in your system PATH. Test by running the command in a normal terminal.
*   **GUI PATH differences**: If Burp is launched from a GUI shortcut, PATH can be truncated. The extension attempts to capture the login shell PATH, but if the backend still doesn't appear, set the full CLI path in the settings.
*   **Authentication**: Most CLI tools require an initial login:
    *   Gemini: `gemini auth login`
    *   Claude: Set `ANTHROPIC_API_KEY` or run `claude login`
    *   Codex: Set `OPENAI_API_KEY`
    *   OpenCode: Configure provider credentials
*   **OpenCode default model**: If you see `AI backend error (opencode-cli): process has not exited`, set a default model in `opencode.json` or add `--model` to the OpenCode command.
*   **Model Name**: Ensure the model name in your command or settings is correct (e.g., `llama3` vs `llama3.1`).
*   **Immediate crash**: If the backend crashes immediately after launch, check the extension output tab for the exit code and error output.

### Empty or "I don't know" responses
*   **Context Limit**: If you attached too many requests, the model might be overwhelmed. Try "Analyze this request" instead of "Find Vulnerabilities".
*   **Ollama not running**: Ensure the Ollama server is running (`ollama serve`).
*   **Model not downloaded**: For Ollama, verify the model is available (`ollama list`).
*   **Wrong backend selected**: Check that the backend dropdown in the top bar matches your intended backend.

### Backend won't start
*   **Auto-restart disabled**: If the backend crashed previously, auto-restart may have been suppressed. Try restarting manually.
*   **Port conflict** (HTTP backends): Ensure the Ollama/LM Studio port is not used by another process.
*   **Environment variables**: Ensure API keys are set in the environment where Burp Suite is running, not just in your terminal profile.
*   **Windows long command line**: Some CLI backends can fail if the prompt/context is extremely large (OS command length limit). Use a smaller selection or switch to an HTTP backend.

### Windows + WSL: Codex CLI in WSL
If Burp runs on Windows but you want Codex to run inside WSL on the same machine, point Burp to a Windows `.cmd` wrapper that forwards the args into WSL.

1. Create a wrapper file, e.g. `D:\Docs\BugBounty\Burp\codex\codex.cmd`:

    ```bat
    @echo off
    setlocal
    wsl.exe -d Ubuntu -- script -q -c "bash -lc 'codex %*'" /dev/null 2>nul | findstr /V /R /C:"^OpenAI Codex v" /C:"^--------" /C:"^workdir:" /C:"^model:" /C:"^provider:" /C:"^approval:" /C:"^sandbox:" /C:"^reasoning" /C:"^session id:" /C:"^user$" /C:"^mcp startup:" /C:"^thinking$"
    exit /b %ERRORLEVEL%
    ```

2. In Burp: **Burp AI Agent → AI Backend tab in the bottom settings panel**

| Setting               | Value                                                                 |
| --------------------- | --------------------------------------------------------------------- |
| **Preferred Backend** | `Codex CLI`                                                           |
| **Codex CLI Command** | `D:\Docs\BugBounty\Burp\codex\codex.cmd chat`                          |

Notes:

* Replace `Ubuntu` with your WSL distro name.
* Ensure Codex is installed and authenticated inside WSL.
* Set `OPENAI_API_KEY` in the WSL environment (not only in Windows) if you use API-key auth.

### Windows: OpenCode CLI not found
*   If OpenCode (or any npm-based CLI) was installed via npm, Burp may not inherit your shell PATH. Set the full path to the npm shim using double backslashes, for example `C:\\Users\\<you>\\AppData\\Roaming\\npm\\opencode.cmd`.
*   If OpenCode was installed via npm, the launcher is `opencode` (without `.exe`). The extension normalizes `opencode.exe`, but a custom path should point to the actual launcher.

## MCP Issues

### MCP Toggle won't stay ON
*   **Port Conflict**: The default port `9876` might be used by another app. Try changing it in **MCP Server tab in the bottom settings panel**.
*   **Bind Error**: Check the Burp extension output for "Address already in use".
*   **Previous instance**: If Burp crashed, a previous MCP server may still be bound to the port. The extension will attempt to shut it down automatically.

### Claude Desktop won't connect
*   **Token**: If **External Access** is enabled, ensure your client sends `Authorization: Bearer <token>`. Localhost access does not require a token.
*   **Config path**: Verify the config file location:
    *   macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
    *   Windows: `%APPDATA%\Claude\claude_desktop_config.json`
*   **HTTPS/TLS**: If you enabled TLS, ensure the client supports the certificate or use the HTTP URL.
*   **Restart Claude Desktop**: After modifying the config, restart Claude Desktop completely.
*   **Port mismatch**: Ensure the port in the config matches the port in the extension settings.

### MCP tools not appearing
*   **Tool gating**: Many tools are disabled by default. Check the **Burp Integration** tab tool toggles in the bottom settings panel.
*   **Unsafe tools**: Enable the "Unsafe Tools" master switch if you need tools that send traffic or modify state.
*   **Pro-only tools**: Scanner-related tools require Burp Suite Professional.

## Scanner Issues

### No issues being created
*   **Confidence threshold**: Passive findings only become issues when the confidence score is >= 85%. Lower-confidence findings appear in the View Findings panel.
*   **Scope**: Ensure the target is in your Burp Suite **Target → Scope**. The scanner defaults to "Scope Only" for safety.
*   **Min severity**: Check that the minimum severity threshold isn't set too high.
*   **Backend not running**: The scanner needs an active backend to analyze requests.

### Active scanner not scanning
*   **Enable toggle**: Ensure the Active toggle is ON in the top bar.
*   **Queue empty**: Requests must be queued via context menu or auto-queue from passive.
*   **Deduplication**: The scanner skips targets scanned within the last hour.
*   **Scope Only**: If enabled, only in-scope targets are scanned.

### AI is too slow
*   Local models (Ollama/LM Studio) performance depends on your hardware. Using a GPU is highly recommended.
*   Check the **Rate Limit** in scanner settings; if set too high, the scanner waits between requests.
*   Reduce **Max Size (KB)** to send smaller context to the AI.
*   Consider using a faster model (e.g., Gemini Flash, GPT-4o Mini, Llama 8B).

## General Issues

### Extension output is empty
*   Ensure Burp Suite was launched from a terminal to see stdout/stderr output.
*   Check **Extensions → Installed → Output/Errors** tabs.

### Settings not saving
*   Settings are stored in Burp's preferences. Ensure you don't have a read-only Burp project file.
*   Some settings require an extension restart to take effect.

### High memory usage
*   Large response bodies can increase memory usage. Reduce **Max Size (KB)** in scanner settings.
*   Disable the passive scanner when not actively needed.
*   Close unused chat sessions.


### Active scan queued 0 targets

If an active scan action reports that no targets were queued:

- The target may be out of scope while **Scope Only** is enabled.
- The request may be invalid for active extraction.
- The active queue may have reached its hard cap (`2000` pending targets).

Check scanner status in the **AI Active Scanner** tab and clear queue if needed.

### Agent profile warning about MCP tools

If Settings shows profile warnings about missing/disabled tools:

- The selected AGENTS profile references MCP tools currently disabled in **Burp Integration**.
- The tool may be gated by **Unsafe mode**.
- The tool may be unavailable in your Burp edition (Pro-only tools).

Enable required tools/toggles or adjust the profile content.
