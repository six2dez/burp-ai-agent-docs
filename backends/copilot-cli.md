# Copilot CLI

Use GitHub Copilot CLI for multi-model cloud analysis powered by GitHub's AI infrastructure. Copilot CLI supports Claude, GPT, and Gemini model families through a single tool.

## Requirements

* `copilot` CLI installed.
* GitHub authentication completed (`copilot auth login` or active GitHub session).

## Setup

1. Install CLI:

```bash
brew install copilot-cli
```

2. Authenticate:

```bash
copilot auth login
```

3. Verify locally:

```bash
copilot -p "hello"
```

4. Configure in **AI Backend** settings tab.

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `Copilot CLI` |
| **Copilot CLI Command** | `copilot` or full path |

Model examples:

```bash
copilot --model claude-sonnet-4.6
copilot --model gpt-5.2
copilot --model gemini-3-pro-preview
```

Available models include: `claude-sonnet-4.6`, `claude-sonnet-4.5`, `claude-haiku-4.5`, `claude-opus-4.6`, `gpt-5.2`, `gpt-5.1-codex`, `gpt-5-mini`, `gemini-3-pro-preview`, and others.

## Notes

### Large Prompt Fallback

For very large prompts (threshold: `32000` chars), the extension uses a file-based fallback path instead of direct argument passing. This reduces CLI argument/STDIN size failures on long contexts.

### Non-Interactive Mode

The extension invokes Copilot CLI with `-p` (non-interactive) flag, which executes the prompt and exits after completion.

### Windows

npm-installed CLI shims are resolved automatically on Windows. The extension detects `.cmd` siblings and uses them instead of shell script shims that Java cannot execute directly.

## Troubleshooting

{% hint style="tip" %}
* `command not found`: use full binary path (e.g., `/opt/homebrew/bin/copilot`).
* Windows: npm shim paths are resolved automatically. If auto-resolution fails, use the full `.cmd` path.
* Auth issues: re-run `copilot auth login` or verify GitHub session.
* Empty output: check Burp extension output/errors tabs and model flag validity.
* Model errors: verify model name matches available choices with `copilot --help`.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
