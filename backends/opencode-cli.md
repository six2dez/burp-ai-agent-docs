# OpenCode CLI

OpenCode is an open-source CLI that supports multiple AI providers through a unified interface. It allows you to use models from Anthropic, OpenAI, Google, and other providers with a single tool.

## Setup

1.  **Install the CLI**: Follow the [OpenCode documentation](https://github.com/opencode-ai/opencode).
2.  **Set API keys** for your chosen provider:
    ```bash
    # For Anthropic models:
    export ANTHROPIC_API_KEY="sk-ant-..."
    # For OpenAI models:
    export OPENAI_API_KEY="sk-..."
    # For Google models:
    export GOOGLE_API_KEY="..."
    ```
3.  **Choose a default model**: Either set a default model in your `opencode.json` (recommended) or pass `--model` in the command. Without a default model, OpenCode can fail with `process has not exited`.
4.  **Verify it works**: Test in your terminal:
    ```bash
    opencode "hello"
    ```
5.  **Configure in Burp**: Open **Burp AI Agent → AI Backend tab in the bottom settings panel** and set:

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `OpenCode CLI` (select from dropdown) |
| **OpenCode CLI Command** | `opencode --model anthropic/claude-sonnet-4-5` |

The model flag must be included in the command field unless you set a default model in `opencode.json`. Examples:

```
opencode --model anthropic/claude-sonnet-4-5
opencode --model openai/gpt-4o
opencode --model google/gemini-1.5-pro
```

## Troubleshooting

*   **"command not found"**: Ensure `opencode` is installed and in your system PATH.
*   **Windows (npm install + PATH issues)**: If Burp cannot find `opencode`, set the full path to the npm shim using double backslashes, for example `C:\\Users\\<you>\\AppData\\Roaming\\npm\\opencode.cmd`. Burp may not inherit your shell PATH.
*   **Windows `.exe`**: If installed via npm, use `opencode` (without `.exe`). The extension normalizes bare `.exe` names to the correct launcher.
*   **Provider errors**: Verify the API key for your chosen provider is configured correctly.
*   **Model not available**: Check that the model identifier follows the `provider/model` format.
*   **"AI backend error (opencode-cli): process has not exited"**: Set a default model in `opencode.json` or add `--model` to the command field.
