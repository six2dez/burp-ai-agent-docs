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

### Model Configuration

If no default model is configured and no `--model` is provided, embedded runs can fail with `process has not exited`.

### Output Parsing

The extension filters OpenCode CLI output to extract the actual AI response:

* **Status/metadata lines**: All lines starting with `> ` (under 120 characters) are treated as OpenCode status output and filtered. This includes `> thinking`, `> loading`, `> building`, and similar status messages.
* **Prompt echo removal**: Only long prompt lines (40+ characters) are used for dedup filtering, preventing false positives where short common terms (e.g., "SQL Injection", "Analyze") would incorrectly match real response content.
* **Version banners**: Lines starting with `opencode v` or `session:` are filtered.

### Idle Timeout

The extension waits up to 30 seconds of idle output before considering the OpenCode process complete. This allows sufficient time for model inference on large prompts without premature termination.

### Windows

npm-installed CLI shims are resolved automatically on Windows. The extension detects `.cmd` siblings and uses them instead of shell script shims that cannot be executed directly by Java.

## Troubleshooting

{% hint style="tip" %}
* `command not found`: use full binary/shim path.
* Windows: npm shim paths are resolved automatically. If auto-resolution fails, use the full `.cmd` path: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\opencode.cmd`.
* Provider errors: verify the API key for selected provider.
* `process has not exited`: configure a default model or add `--model`.
* Blank responses: check that the model is producing output by running the same command in a terminal. If the model takes longer than 30 seconds to start producing output, the process may time out.
* Unrendered responses: ensure the response is not being filtered by the output parser. Check the extension output tab for raw process output.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
