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

For prompts above `32_000` characters (the internal `LARGE_PROMPT_THRESHOLD`), the extension avoids passing the full text on the command line. Instead it:

1. Writes the combined prompt to a temp file named `burp_uv_prompt_*.txt` with POSIX permissions `0600` where supported.
2. Invokes Claude with an instruction such as *"Please process the instructions and data provided in the following file: &lt;path&gt;"*, passing the path as an argument.
3. Deletes the temp file in a `finally` block after the response is consumed.

This avoids OS-level argv/stdin length limits on long contexts while keeping the prompt on-host.

### Session Resume

Claude CLI sessions are kept sticky across turns:

* The first turn is launched with `--session-id <uuid>`.
* Subsequent turns on the same chat session use `--resume <uuid>` so conversation history is reconstructed inside Claude rather than replayed by the extension. The session ID is held in a thread-safe `AtomicReference` with CAS updates.

If the process is killed or `Clear Chat` is used, the session ID is released and a new `--session-id` is minted on the next turn.

### Windows

npm-installed CLI shims are resolved automatically on Windows. The extension detects `.cmd` siblings and uses them instead of shell script shims that Java cannot execute directly.

### Windows + WSL bridge

If Burp runs on Windows and Claude CLI is installed inside WSL, the simplest setup is to point **Claude CLI Command** at a one-liner that invokes WSL with an interactive shell so `PATH` and auth env vars from `~/.bashrc` / `~/.profile` are loaded:

```
wsl -d Debian bash -ic "/home/<username>/.local/bin/claude"
```

Notes:

* Replace `Debian` with the name of your distro (`wsl -l -v` lists installed ones) and `<username>` with your WSL user.
* `bash -ic` runs an **i**nteractive shell that sources rc files, so `ANTHROPIC_API_KEY` (or whatever `claude login` wrote) is picked up; `-c` then executes the binary.
* Use the **absolute path** to the `claude` binary inside WSL (`which claude` from WSL prints it). Relying on bare `claude` can fail if the rc files don't put `~/.local/bin` on `PATH` for non-login shells.
* No `.cmd` wrapper file is required — the extension passes its CLI args through the same way it does for native commands.

If you prefer a wrapper script instead of a single command (e.g. to filter banner output, switch distros, or share the config across machines), the `.cmd` pattern documented for [Codex CLI](codex-cli.md#windows--wsl-bridge) works the same way for Claude.

## Troubleshooting

{% hint style="tip" %}
* `command not found`: use full binary path.
* Windows: npm shim paths are resolved automatically. If auto-resolution fails, use the full `.cmd` path: `C:\\Users\\<you>\\AppData\\Roaming\\npm\\claude.cmd`.
* Auth issues: re-run `claude login` or verify `ANTHROPIC_API_KEY`.
* Empty output: check Burp extension output/errors tabs and model flag validity.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
