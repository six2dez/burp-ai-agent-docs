# Codex CLI

OpenAI's Codex CLI provides access to GPT-4o and other OpenAI models. It is the default backend and offers reliable, versatile performance for general security analysis.

## Setup

1.  **Install the CLI**: Follow the official [OpenAI Codex documentation](https://github.com/openai/codex).

    ```bash
    npm install -g @openai/codex
    ```
2.  **Set your API key**:

    ```bash
    export OPENAI_API_KEY="sk-..."
    ```
3.  **Verify it works**: Test in your terminal:

    ```bash
    codex "hello"
    ```
4. **Configure in Burp**: Open **Burp AI Agent → AI Backend tab in the bottom settings panel** and set:

| Setting               | Value                              |
| --------------------- | ---------------------------------- |
| **Preferred Backend** | `Codex CLI` (select from dropdown) |
| **Codex CLI Command** | `codex chat` (default)             |

To specify a model, include the flag in the command field:

| Setting               | Value                   |
| --------------------- | ----------------------- |
| **Codex CLI Command** | `codex --model gpt-5.2` |

## Windows + WSL (Codex CLI in WSL)

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
* If you use this setup, also see **reference/troubleshooting.md** for the full Windows + WSL walkthrough.

## Troubleshooting

* **"command not found"**: Ensure `codex` is installed and in your system PATH.
* **Windows (npm install + PATH issues)**: If Burp cannot find `codex`, set the full path to the npm shim using double backslashes, for example `C:\\Users\\<you>\\AppData\\Roaming\\npm\\codex.cmd`. Burp may not inherit your shell PATH.
* **Authentication errors**: Verify your `OPENAI_API_KEY` environment variable is set.
* **Rate limiting**: OpenAI applies rate limits based on your plan tier. Check usage at [platform.openai.com](https://platform.openai.com).
