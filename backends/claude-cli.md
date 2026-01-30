# Claude CLI

Anthropic's Claude CLI provides access to the Claude family of models, which perform well at reasoning, code analysis, and structured output generation.

## Setup

1.  **Install the CLI**: Follow the official [Anthropic documentation](https://docs.anthropic.com/en/docs/claude-code).
    ```bash
    npm install -g @anthropic-ai/claude-code
    ```
2.  **Authenticate**: Either set the environment variable or run login:
    ```bash
    export ANTHROPIC_API_KEY="sk-ant-..."
    # or
    claude login
    ```
3.  **Verify it works**: Test in your terminal:
    ```bash
    claude "hello"
    ```
4.  **Configure in Burp**: Open **Burp AI Agent → Settings → AI Backend** and set:

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `Claude CLI` (select from dropdown) |
| **Claude CLI Command** | `claude` (or the full path, e.g., `/usr/local/bin/claude`) |

To specify a model, include the flag in the command field:

| Setting | Value |
| :--- | :--- |
| **Claude CLI Command** | `claude --model sonnet` |

## Troubleshooting

*   **"command not found"**: Ensure `claude` is installed and in your system PATH.
*   **Windows (npm install + PATH issues)**: If Burp cannot find `claude`, set the full path to the npm shim using double backslashes, for example `C:\\Users\\<you>\\AppData\\Roaming\\npm\\claude.cmd`. Burp may not inherit your shell PATH.
*   **Authentication errors**: Verify your `ANTHROPIC_API_KEY` is set correctly, or re-authenticate with `claude login`.
*   **Empty responses**: Check the Burp extension output tab for error details. The model name may be incorrect.
