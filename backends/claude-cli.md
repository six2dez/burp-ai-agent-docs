# Claude CLI

Use Claude CLI when you want strong reasoning and structured output from Anthropic models.

## Requirements

* `claude` CLI installed.
* Authentication completed (`ANTHROPIC_API_KEY` or `claude login`).

## Setup

1. Install CLI:

```bash
npm install -g @anthropic-ai/claude-code
```

2. Authenticate:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
# or
claude login
```

3. Verify locally:

```bash
claude "hello"
```

4. Configure in **AI Backend** settings tab.

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `Claude CLI` |
| **Claude CLI Command** | `claude` or full path |

Model example:

```bash
claude --model sonnet
```

## Notes

### Large Prompt Fallback

For very large prompts (threshold: `32000` chars), the extension uses a file-based fallback path instead of direct argument passing. This reduces CLI argument/STDIN size failures on long contexts.

## Troubleshooting

{% hint style="tip" %}
* `command not found`: use full binary path.
* Windows npm shim example: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\claude.cmd`.
* Auth issues: re-run `claude login` or verify `ANTHROPIC_API_KEY`.
* Empty output: check Burp extension output/errors tabs and model flag validity.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
