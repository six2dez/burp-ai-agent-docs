# OpenCode CLI

OpenCode CLI offers a single CLI with multiple provider backends.

## Requirements

* `opencode` installed.
* Provider credentials (Anthropic/OpenAI/Google/etc).
* Default model configured or explicit `--model` in command.

## Setup

1. Install OpenCode CLI.
2. Set provider API keys.
3. Verify local command works.
4. Configure in **AI Backend** settings tab.

Example environment setup:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
export GOOGLE_API_KEY="..."
```

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `OpenCode CLI` |
| **OpenCode CLI Command** | `opencode --model anthropic/claude-sonnet-4-5` |

Other model examples:

```text
opencode --model openai/gpt-4o
opencode --model google/gemini-1.5-pro
```

## Notes

If no default model is configured and no `--model` is provided, embedded runs can fail with `process has not exited`.

## Troubleshooting

{% hint style="tip" %}
* `command not found`: use full binary/shim path.
* Windows npm shim example: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\opencode.cmd`.
* Provider errors: verify the API key for selected provider.
* `process has not exited`: configure a default model or add `--model`.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
