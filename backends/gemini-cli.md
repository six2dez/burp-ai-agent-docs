# Gemini CLI

Gemini CLI is useful for large-context cloud analysis and agent-style workflows.

## Requirements

* `gemini` CLI installed.
* Authentication completed with `gemini auth login`.

## Setup

1. Install/verify the CLI from the official docs.
2. Authenticate:

```bash
gemini auth login
```

3. Verify locally:

```bash
gemini "hello"
```

4. Configure in **AI Backend** settings tab.

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `Gemini CLI` |
| **Gemini CLI Command** | `gemini --output-format text --model gemini-2.5-flash --yolo` |

## Notes

`--yolo` is used by default to avoid interactive approval prompts that can block embedded MCP workflows.

## Troubleshooting

{% hint style="tip" %}
* `command not found`: use full executable path if needed.
* Windows npm shim example: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\gemini.cmd`.
* `ModelNotFoundError` / `RESOURCE_EXHAUSTED`: choose available model and update `--model`.
* Auth errors: re-run `gemini auth login`.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
