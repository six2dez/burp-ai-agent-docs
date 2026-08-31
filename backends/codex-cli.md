# Codex CLI

Codex CLI provides OpenAI model access for general security analysis and code-oriented workflows.

## Requirements

* `codex` CLI installed.
* Codex authentication completed (`codex login` or an API-key/configuration method supported by your installed Codex version).

## Setup

1. Install CLI:

```bash
npm install -g @openai/codex
```

2. Authenticate. For an interactive login:

```bash
codex login
```

3. Verify locally:

```bash
codex exec "hello"
```

4. Configure in **AI Backend** settings tab.

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `codex-cli` |
| **Codex CLI Command** | `codex chat` (default) |

Model example:

```bash
codex --model gpt-5.2
```

`codex chat` is retained as the persisted default for compatibility. At dispatch time the adapter removes the legacy `chat` token and invokes `codex exec --color never --skip-git-repo-check`, feeds the prompt on stdin, and reads the final answer through a temporary `--output-last-message` file. That file is deleted in the call's cleanup path; a bounded shutdown registry retries cleanup when Burp exits while a call is still in flight.

## Notes

### Windows

The extension automatically resolves npm-installed CLI shims on Windows. When `codex` is configured, the resolver:

1. Checks for a `.cmd` sibling next to the resolved path (e.g., `codex.cmd` alongside `codex`).
2. Falls back to wrapping the command with `cmd /c` if no `.cmd` sibling is found.

This prevents `CreateProcess error=193` that occurs when Java tries to execute a shell script shim directly on Windows.

{% hint style="info" %}
You do not need to manually specify `.cmd` extensions. The extension handles this automatically for all npm-installed CLI backends.
{% endhint %}

### Windows + WSL bridge

If Burp runs on Windows and Codex runs in WSL, set **Codex CLI Command** to a `.cmd` wrapper that forwards args into WSL.

Wrapper example:

```bat
@echo off
setlocal
wsl.exe -d Ubuntu -- script -q -c "bash -lc 'codex %*'" /dev/null 2>nul | findstr /V /R /C:"^OpenAI Codex v" /C:"^--------" /C:"^workdir:" /C:"^model:" /C:"^provider:" /C:"^approval:" /C:"^sandbox:" /C:"^reasoning" /C:"^session id:" /C:"^user$" /C:"^mcp startup:" /C:"^thinking$"
exit /b %ERRORLEVEL%
```

## Troubleshooting

{% hint style="tip" %}
* `command not found`: use full binary path or npm shim path.
* Windows: npm shim paths are resolved automatically. If auto-resolution fails, use the full `.cmd` path: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\codex.cmd`.
* Auth errors: run `codex login` from the same OS account that launches Burp, or verify the API-key/configuration method used by your Codex installation is visible to the Burp process.
* Rate limits: check provider quota/tier.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
