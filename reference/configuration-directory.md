# Configuration Directory

The extension stores all runtime state under a single directory in the user's home folder. The directory keeps the legacy `burp-ai-agent` name for upgrade compatibility — see [Legacy Identifier Reference](#legacy-identifier-reference) below for why the product name and the on-disk name diverge.

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

## Legacy Identifier Reference

The product is called **Custom AI Agent** but several internal identifiers still use the legacy name `burp-ai-agent`. This table lists exactly where each form appears.

| Context | Value | Notes |
| :--- | :--- | :--- |
| Product and Burp extension name | `Custom AI Agent` | Visible in Burp's Extensions list and in the main tab. |
| Release artifact (full build) | `Custom-AI-Agent-full-<version>.jar` | Published on GitHub Releases; registers all 59 MCP tools. Built with `./gradlew shadowJar`. |
| Release artifact (BApp Store build) | `Custom-AI-Agent-<version>.jar` | BApp Store listing; registers only the 8 extension-native AI MCP tools. Built with `./gradlew shadowJar -PstoreBuild=true`. |
| Checksum / SBOM | `Custom-AI-Agent-<version>.jar.sha256`, `bom.json` | Attached to every GitHub release. |
| Burp tab title | `AI Agent` | Short form; does not include the word "Custom". |
| GitHub repository | `github.com/six2dez/burp-ai-agent` | Repo URL kept to avoid breaking external links, issues, and forks. |
| Runtime directory | `~/.burp-ai-agent/` (Windows: `%USERPROFILE%\.burp-ai-agent\`) | Documented above. |
| Java package | `com.six2dez.burp.aiagent` | Kept to preserve backend SPI compatibility for drop-in JARs. |
| Gradle project name | `burp-ai-agent` (in `settings.gradle.kts`) | Internal only. |
| MCP implementation string | `burp-ai-agent` | Shows up in MCP `initialize` responses; clients may rename. |
| MCP server name in client configs (examples) | `burp-ai-agent` | Suggested key in `claude_desktop_config.json` and similar; you can call it anything you like. |
| Audit log path | `~/.burp-ai-agent/audit.jsonl` | Inside the runtime directory above. |

### Why the Two Names Coexist

The product is called **Custom AI Agent** because PortSwigger naming guidance asks third-party extensions to avoid using "Burp" as the leading word in a product name. User-facing strings and the JAR artifact carry the new name; internal identifiers still use the legacy `burp-ai-agent` form on purpose, for three reasons:

1. **Migrations**: existing users already have `~/.burp-ai-agent/` populated with audit logs, prompt caches, agent profiles, and drop-in backend JARs. Renaming the directory would invalidate all of that silently.
2. **SPI compatibility**: external drop-in backends ship JARs that register under `com.six2dez.burp.aiagent.backends.AiBackendFactory` via `META-INF/services`. Renaming the package would break every published third-party backend.
3. **GitHub identifiers**: repo URL, issue IDs, and pull requests are stable references that appear in pinned dependencies, automation, and user bookmarks.

### What to Expect in Each Surface

* **UI, chat labels, audit payloads, documentation titles** → `Custom AI Agent`.
* **Filesystem paths, repo URL, Java imports, Gradle tasks** → `burp-ai-agent`.
* **MCP client configs** → either is fine; the identifier is advisory. Prefer `burp-ai-agent` if you want copy-paste examples from the docs to keep working.

### Ops and CI Naming Notes

When wiring scripts, automation, or dashboards against this extension:

* Scripts that download or pin the release JAR should target `Custom-AI-Agent-*.jar`.
* Dashboards or Slack messages that reference the Burp tab should use the short form **`AI Agent`** (the user-visible tab name).

Scripts that reference `~/.burp-ai-agent/`, the GitHub repo URL, the Java package, or the MCP implementation string follow the legacy identifier form — those do **not** need to match the product name.

## Related Pages

* [Settings Reference](settings-reference.md)
* [Audit Logging](../privacy/audit-logging.md)
* [AI Request Logger](../privacy/ai-request-logger.md)
* [Agent Profiles](../user-guide/agent-profiles.md)
