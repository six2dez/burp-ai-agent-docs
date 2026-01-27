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
4.  **Configure in Burp**: Open **Burp AI Agent → Settings → AI Backend** and set:

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `Codex CLI` (select from dropdown) |
| **Codex CLI Command** | `codex chat` (default) |

To specify a model, include the flag in the command field:

| Setting | Value |
| :--- | :--- |
| **Codex CLI Command** | `codex --model gpt-4.1` |

## Launch Modes

The Codex backend supports two launch modes:

*   **Embedded (Built-in Chat)**: The extension manages the CLI process internally. Responses stream directly into the chat panel.
*   **External Terminal**: Opens the CLI in your system terminal (iTerm2, cmd.exe, etc.). Useful for debugging or when you prefer the native CLI experience.

Configure the launch mode via **Settings → AI Backend → External Terminal Template**.

## Troubleshooting

*   **"command not found"**: Ensure `codex` is installed and in your system PATH.
*   **Authentication errors**: Verify your `OPENAI_API_KEY` environment variable is set.
*   **Rate limiting**: OpenAI applies rate limits based on your plan tier. Check usage at [platform.openai.com](https://platform.openai.com).
