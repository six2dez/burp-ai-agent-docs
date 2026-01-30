# Gemini CLI

Google's Gemini CLI provides access to the Gemini family of models, which support large context windows (1M+ tokens) and multimodal input.

## Setup

1. **Install the CLI**: Follow the official [Google Gemini CLI documentation](https://geminicli.com/docs/).
2.  **Authenticate**: Run this in your terminal:

    ```bash
    gemini auth login
    ```
3.  **Verify it works**: Test in your terminal:

    ```bash
    gemini "hello"
    ```
4. **Configure in Burp**: Open **Burp AI Agent → Settings → AI Backend** and set:

| Setting                | Value                                                      |
| ---------------------- | ---------------------------------------------------------- |
| **Preferred Backend**  | `Gemini CLI` (select from dropdown)                        |
| **Gemini CLI Command** | `gemini` (or the full path, e.g., `/usr/local/bin/gemini`) |

To specify a model, include the flag in the command field:

| Setting                | Value                             |
| ---------------------- | --------------------------------- |
| **Gemini CLI Command** | `gemini --model gemini-2.0-flash` |

## Troubleshooting

* **"command not found"**: Ensure `gemini` is installed and in your system PATH. Try using the full path to the binary.
* **Windows (npm install + PATH issues)**: If Burp cannot find `gemini`, set the full path to the npm shim using double backslashes, for example `C:\\Users\\<you>\\AppData\\Roaming\\npm\\gemini.cmd`. Burp may not inherit your shell PATH.
* **Authentication errors**: Run `gemini auth login` in your terminal and follow the OAuth flow.
* **Rate limiting**: The free tier has rate limits. Consider upgrading to a paid tier for heavy scanning use.
