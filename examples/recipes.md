# Recipes: How Do I…

Short, copy-pasteable answers to common operational questions. Each recipe is self-contained and links to the canonical page for deeper context.

## …rotate the MCP bearer token?

When the **External Access** path is exposed, the bearer token in **Settings → MCP Server → Token** acts as the only credential. Rotate it when sharing a workstation, after a suspected leak, or on a schedule.

1. Open **Settings → MCP Server**.
2. Click **Regenerate** next to **Token**. Clearing the editable field does not generate a replacement automatically.
3. Hit **Save**. The MCP server restarts automatically.
4. Push the new value to every external client that connects (Claude Desktop config, gateway env var, etc.).

Cross-reference: [MCP Security Model](../mcp/security-model.md).

## …rotate the MCP TLS certificate?

The auto-generated keystore at `~/.burp-ai-agent/certs/mcp-keystore.p12` is valid for 365 days. To force regeneration without waiting:

```bash
# Stop Burp (or disable MCP) first.
rm ~/.burp-ai-agent/certs/mcp-keystore.p12
```

Re-enable MCP. With **Auto-Generate Certificate** on, the next start regenerates the keystore (RSA 2048, SHA256withRSA, 365 days, `CN=burp-mcp`) and stores a fresh password in Burp preferences.

If you maintain a custom keystore, replace the file and re-set **Keystore Password** under **Settings → MCP Server**.

Cross-reference: [Configuration Directory](../reference/configuration-directory.md).

## …filter the audit log for a specific trace?

Trace correlation is stored by the AI Request Logger. If rolling persistence is enabled, pull one trace from its active JSONL file with:

```bash
jq -c 'select(.metadata.traceId == "scanner-job-12345")' \
  ~/.burp-ai-agent/logs/ai-request-log.jsonl
```

The separate audit stream does not put a trace ID on every event. It can still filter prompt launch metadata, for example a specific custom-prompt source:

```bash
jq 'select(.type == "prompt" and .payload.promptSource == "CUSTOM_SAVED")' \
  ~/.burp-ai-agent/audit.jsonl
```

Cross-reference: [Audit Logging](../privacy/audit-logging.md).

## …clear the prompt cache for one engagement?

Each project has its own cache subdirectory. Its name is the first eight characters of `api.project().id()` (or `default` if that API lookup fails). Identify the directory by time/size while the relevant project is open, disable the passive scanner, and remove only that subdirectory:

```bash
ls ~/.burp-ai-agent/cache/
# Example output:
# 5f3a2c-acme-internal
# 9b71fd-public-bb-target

rm -rf ~/.burp-ai-agent/cache/9b71fd-public-bb-target
```

The plugin recreates the directory on the next cache write. Other projects' caches are untouched.

Cross-reference: [Passive AI Scanner → Cache Behavior](../scanners/passive.md#cache-key-prompt-hash).

## …force a fresh passive-scanner AI call?

There is no per-request cache-bypass switch. Three independent gates can reuse or skip work: endpoint dedup, response-fingerprint dedup, and the in-memory/persistent prompt-result cache. Changing a header alone does not bypass the earlier endpoint gate.

For a controlled fresh run, disable the passive scanner, turn **Persistent cache** off, set **Endpoint dedup**, **Response dedup**, and **Prompt cache TTL** to `1` minute, then reload the extension (or restart Burp) to clear the in-memory maps. Re-enable the scanner, wait past the one-minute window if the request was already seen, run the request once, and restore the prior settings. If persistent cache must remain enabled, remove only the current project's cache directory while the scanner is disabled as described above.

## …connect an external MCP client?

For Claude Desktop and any client that speaks MCP over SSE, the canonical URL is `http://127.0.0.1:9876/sse`. Loopback works without auth; external access requires bearer + TLS.

Example `claude_desktop_config.json` fragment:

```json
{
  "mcpServers": {
    "burp-ai-agent": {
      "command": "npx",
      "args": ["-y", "supergateway", "--sse", "http://127.0.0.1:9876/sse"]
    }
  }
}
```

For external access, replace `127.0.0.1` with the bound IP, switch to `https://`, and pass the bearer:

```bash
supergateway --sse https://<host>:9876/sse \
  --header "Authorization: Bearer <token>"
```

Cross-reference: [MCP Overview](../mcp/overview.md), [Burp Scan Skill (Terminal AI)](burp-scan-skill.md).

## …migrate agent profiles between machines?

Profiles live as plain Markdown files in `~/.burp-ai-agent/AGENTS/` plus a `default` marker file naming the active one.

```bash
# On the source machine
tar czf agents.tar.gz -C ~/.burp-ai-agent AGENTS

# On the target machine
mkdir -p ~/.burp-ai-agent
tar xzf agents.tar.gz -C ~/.burp-ai-agent

# Restart Burp or re-open the AI Backend settings tab to pick up the profile list.
```

Cross-reference: [Agent Profiles](../user-guide/agent-profiles.md).

## …export the saved custom prompt library?

Open **Settings → Custom Prompts → Export JSON**. The output is pretty-printed and contains every entry's `id`, `title`, `promptText`, `tags`, `showInContextMenu`, and `isFavorite`. Drop the file on another workstation and use **Import JSON** to merge by `id`.

Cross-reference: [Settings Reference → Custom Prompts](../reference/settings-reference.md#custom-prompts), [Context Menus → Custom Prompt Library](../user-guide/context-menus.md#custom-prompt-library).

## …switch privacy mode mid-engagement without leaking the previous mode's data?

Privacy mode applies to the prompt that is *about to* be built, not retroactively. So:

1. Switch **Privacy Mode** under **Settings → Privacy & Logging**.
2. Confirm the top-bar pill reflects the new mode.
3. Re-trigger the analysis. The next context capture is redacted under the new mode.

Anything already in the audit log or already in flight keeps the representation produced under the old mode — including unredacted content if that mode was `OFF`. There is no rewrite. If that is a problem, protect or remove the relevant artifact according to the engagement's retention policy, clear the relevant cache state, and rerun under the new mode.

Cross-reference: [Privacy Modes](../privacy/privacy-modes.md), [Best Practices → Privacy Posture by Environment](../reference/best-practices.md#privacy-posture-by-environment).

## …enable rolling persistence for the AI Request Logger?

The in-memory logger keeps the last `500` entries by default. To also persist them to rotating JSONL files, set JVM properties at Burp startup:

```bash
java \
  -Dburp.ai.logger.rolling.enabled=true \
  -Dburp.ai.logger.rolling.maxBytes=2097152 \
  -Dburp.ai.logger.rolling.maxFiles=10 \
  -jar burpsuite.jar
```

Files land in `~/.burp-ai-agent/logs/`. Defaults: `1 MB` per file, `5` rolled files.

Cross-reference: [AI Request Logger](../privacy/ai-request-logger.md), [Settings Reference → Rolling Log Persistence](../reference/settings-reference.md#rolling-log-persistence-jvm-properties).

## …add a custom backend without rebuilding?

Drop the compiled JAR into `~/.burp-ai-agent/backends/` and restart Burp. The plugin picks it up via `ServiceLoader` on the next start.

```bash
mkdir -p ~/.burp-ai-agent/backends
cp ./build/libs/my-custom-backend.jar ~/.burp-ai-agent/backends/
```

After restart the new backend appears in **Settings → AI Backend → Preferred Backend**. If it does not, check **Extensions → Output** for `ServiceLoader` errors from the JAR.

Cross-reference: [Adding a Backend](../developer/adding-backend.md).

## …stop a runaway active scan?

If the active scan queue is consuming traffic budget faster than expected:

1. Toggle **Active** off in the top bar. In-flight requests finish; nothing new is dequeued.
2. Open the **AI Active Scanner** tab and clear the queue from the runtime controls.
3. (Optional) Lower **Max Concurrent Scans** and **Max Risk Level** before re-enabling.

Toggling Active off is non-destructive — confirmed findings already created as Burp issues stay in **Target → Issues**.

Cross-reference: [Active AI Scanner](../scanners/active.md).

## Related Pages

* [Typical Workflows](typical-workflows.md)
* [Sample Prompts](sample-prompts.md)
* [Best Practices & Hardening Checklist](../reference/best-practices.md)
* [Backend Troubleshooting](../backends/troubleshooting.md)
* [Troubleshooting (general)](../reference/troubleshooting.md)
