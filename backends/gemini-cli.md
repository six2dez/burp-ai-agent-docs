# Gemini CLI

Google's Gemini CLI provides access to the Gemini family of models, known for their massive context windows (1M+ tokens) and strong multimodal capabilities.

## Setup

1.  **Install the CLI**: Follow the official [Google Gemini CLI documentation](https://ai.google.dev/gemini-api/docs/cli).
2.  **Authenticate**: Run this in your terminal:
    ```bash
    gemini auth login
    ```
3.  **Verify it works**: Test in your terminal:
    ```bash
    gemini "hello"
    ```
4.  **Configure in Burp**: Open **Burp AI Agent → Settings → AI Backend** and set:

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `Gemini CLI` (select from dropdown) |
| **Gemini CLI Command** | `gemini` (or the full path, e.g., `/usr/local/bin/gemini`) |

To specify a model, include the flag in the command field:

| Setting | Value |
| :--- | :--- |
| **Gemini CLI Command** | `gemini --model gemini-2.0-flash` |

## Recommended Models

| Model | Context Window | Best For |
| :--- | :--- | :--- |
| **Gemini 1.5 Pro** | 1M+ tokens | Complex JS analysis, large request/response pairs, multimodal analysis. |
| **Gemini 1.5 Flash** | 1M+ tokens | Quick summaries, high-volume scanning. Very cost-effective. |
| **Gemini 2.0 Flash** | 1M+ tokens | Latest generation, improved reasoning. |

## Why Gemini for Security?

*   **Massive context window**: Analyze entire JavaScript bundles or large API responses without truncation.
*   **Cost-effective**: Gemini Flash is one of the cheapest cloud options (~$0.15 per 1,000 requests).
*   **Free tier available**: Google provides a free usage tier for Gemini API.

## Troubleshooting

*   **"command not found"**: Ensure `gemini` is installed and in your system PATH. Try using the full path to the binary.
*   **Authentication errors**: Run `gemini auth login` in your terminal and follow the OAuth flow.
*   **Rate limiting**: The free tier has rate limits. Consider upgrading to a paid tier for heavy scanning use.
