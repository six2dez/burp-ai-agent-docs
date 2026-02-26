# Codex CLI

Codex CLI provides OpenAI model access for general security analysis and code-oriented workflows.

## Requirements

* `codex` CLI installed.
* `OPENAI_API_KEY` available in the Burp runtime environment.

## Setup

1. Install CLI:

```bash
npm install -g @openai/codex
```

2. Export API key:

```bash
export OPENAI_API_KEY="sk-..."
```

3. Verify locally:

```bash
codex "hello"
```

4. Configure in **AI Backend** settings tab.

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `Codex CLI` |
| **Codex CLI Command** | `codex chat` (default) |

Model example:

```bash
codex --model gpt-5.2
```

## Notes

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
* Windows npm shim example: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\codex.cmd`.
* Auth errors: verify `OPENAI_API_KEY` in Burp runtime env.
* Rate limits: check provider quota/tier.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
