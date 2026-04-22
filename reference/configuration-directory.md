# Configuration Directory

The extension stores all runtime state under a single directory in the user's home folder. The directory keeps the legacy `burp-ai-agent` name for upgrade compatibility — see [Naming & Paths](naming-and-paths.md).

## Location

| Platform | Path |
| :--- | :--- |
| macOS | `~/.burp-ai-agent/` |
| Linux | `~/.burp-ai-agent/` |
| Windows | `%USERPROFILE%\.burp-ai-agent\` (e.g., `C:\Users\<you>\.burp-ai-agent\`) |

The directory is created on first start. `AGENTS/` is populated with bundled profiles at the same time.

## Layout

```text
~/.burp-ai-agent/
├── audit.jsonl
├── bundles/
├── contexts/
├── cache/
│   └── <projectId>/
│       ├── index.json
│       └── entries/
├── backends/
│   └── my-backend.jar        # optional drop-in
├── AGENTS/
│   ├── default               # plain-text marker with the active profile name
│   ├── pentester.md
│   ├── bughunter.md
│   ├── auditor.md
│   └── <custom>.md           # your profiles
├── certs/
│   └── mcp-keystore.p12
└── logs/                     # created on demand
    ├── ai-request-log.jsonl
    └── ai-request-log.1.jsonl
```

## Entry Reference

| Entry | Created by | Purpose | Contains secrets? |
| :--- | :--- | :--- | :--- |
| `audit.jsonl` | `AuditLogger` when **Audit Logging** is enabled | Append-only JSONL of prompt, scanner, and MCP events with per-event SHA-256 payload hashes. | Yes (redacted prompts and tool results). |
| `bundles/` | `AuditLogger.writePromptBundle` | Frozen `PromptBundle` JSON (and exported ZIP) snapshots of context-driven prompts for reproducibility. | Yes. |
| `contexts/` | `AuditLogger.writeContextFile` | Context JSON files indexed by SHA-256. Referenced from bundles. | Yes. |
| `cache/<projectId>/` | `PersistentPromptCache` | Per-project on-disk cache of scanner AI results. Invalidated by `Response Fingerprint` changes; TTL and max size set in **Passive Scanner** settings. | No (only parsed AI output, which obeys the active privacy mode). |
| `backends/*.jar` | User | Drop-in backend JARs loaded via `ServiceLoader` at startup; see [Adding a Backend](../developer/adding-backend.md). | Depends on the JAR. |
| `AGENTS/*.md` | Extension at first launch + user | Agent profiles. Editable; changes are picked up on the next action without restart. | No. |
| `AGENTS/default` | `AgentProfileLoader` | Plain-text marker file with the active profile name (matches a sibling `*.md`). | No. |
| `certs/mcp-keystore.p12` | `McpTls` auto-generation | PKCS12 keystore for the MCP TLS listener (RSA 2048 / SHA256withRSA / 365 d / `CN=burp-mcp`). | Yes — keystore password is stored in Burp preferences. |
| `logs/*.jsonl` | `AiRequestLogger` when rolling persistence is enabled via JVM properties | Rolling JSONL copies of the in-memory activity log. | Yes. |

## Chat Sessions Are Not Here

Chat sessions do **not** live in this directory. They are stored in Burp's `extensionData()` keyed to the active `.burp` project file, so switching projects switches sessions. See [Chat & Sessions → Project-Scoped Storage](../user-guide/chat-sessions.md#project-scoped-storage).

## Backup and Portability

* **Safe to copy between machines**: `AGENTS/*.md`, `backends/*.jar`, `audit.jsonl` (for archival). These contain no host-specific state.
* **Do not copy between machines**: `cache/<projectId>/`. The cache is indexed by project ID and response fingerprints that include local-only values; promoting a copy from another host produces irrelevant hits.
* **Secrets**: the MCP keystore at `certs/mcp-keystore.p12` and its password (stored in Burp preferences, not here) together authenticate the local MCP listener. Rotate both when a host is retired.

## Cleanup

| Goal | Action |
| :--- | :--- |
| Forget one project's cached analyses | Delete `cache/<projectId>/`. |
| Reset MCP TLS | Delete `certs/mcp-keystore.p12`; the next MCP start will regenerate it. |
| Start fresh for a new client engagement | Move the directory aside (`mv ~/.burp-ai-agent ~/.burp-ai-agent.bak`) before launching Burp. |
| Disable rolling logs | Unset the `-Dburp.ai.logger.rolling.enabled=true` JVM flag at Burp startup (the `logs/` directory remains but no new entries are appended). |

## Related Pages

* [Naming & Paths](naming-and-paths.md)
* [Settings Reference](settings-reference.md)
* [Audit Logging](../privacy/audit-logging.md)
* [AI Request Logger](../privacy/ai-request-logger.md)
* [Agent Profiles](../user-guide/agent-profiles.md)
