# Upgrade & Migration Guide

This page covers how to upgrade between Custom AI Agent releases and what the extension migrates automatically versus what requires a manual action.

## Before Upgrading

1. Back up `~/.burp-ai-agent/audit.jsonl`, `~/.burp-ai-agent/bundles/`, and `~/.burp-ai-agent/AGENTS/` if you rely on them for compliance or workflow reasons.
2. Record the currently active agent profile (shown in **Settings → AI Backend**).
3. Note your chosen **Privacy Mode** — upgrades never change an explicit user choice but may apply a new default to blank settings.

## Release-by-Release Migration Notes

### v0.6.0 (2026-04-21)

Breaking:

* **JAR renamed** — release artifact is now `Custom-AI-Agent-<version>.jar` (was `Burp-AI-Agent-<version>.jar`). Update download URLs, deployment scripts, and pinned symlinks.
* **SHA-256 checksum and CycloneDX SBOM** — every release attaches `*.jar.sha256` and `bom.json`. See [Installation → Verify JAR Integrity](../getting-started/installation.md#verify-jar-integrity-sha-256).
* **Privacy default is now `BALANCED`** — new installs (and anyone who never chose a mode explicitly) get redaction by default. Explicit user choices are preserved.

Automatic migrations:

* MCP proxy-history preprocessing settings are added with their defaults. You'll see the new rows in **Settings → MCP Server** without any action required.
* `VulnClassInventoryTest` locks the canonical 62-class list — the duplicate `RACE_CONDITION` enum was removed; any existing cache entries keyed to it remain valid but will not be reissued.

Manual follow-ups:

* Update any CI scripts that hardcode `Burp-AI-Agent-*.jar`.
* If you relied on the implicit `OFF` privacy mode, re-select it explicitly in **Settings → Privacy & Logging**.

### v0.5.0 (2026-04-02)

Breaking:

* **Chat sessions moved from global `preferences()` to project-scoped `extensionData()`**. First-time open after upgrade migrates existing sessions into the currently active Burp project. Sessions in other projects are not auto-migrated — open each project once to trigger its own migration.
* **Settings schema bumped to v3** — see [Settings Migration](../developer/settings-migration.md).
* **`custom.prompt.library.v1`** preference key introduced for the new custom prompt library.

Automatic migrations:

* Sessions migrate on first open per project.
* `ScanKnowledgeBase` is cleared whenever Burp detects a project change (polled every 30 s) to prevent cross-project leakage.
* The default privacy mode at the time was still `OFF` — if you upgraded through v0.5.0 to v0.6.0, the privacy default bump happens at v0.6.0, not here.

Manual follow-ups: none beyond opening each project you want migrated.

### v0.4.0 (2026-03-06)

Automatic migrations:

* TLS certificate auto-generation was switched from Netty's `SelfSignedCertificate` (incompatible with JDK 25) to JDK's built-in `keytool`. Existing keystores keep working; the change only affects new installs or keystore regeneration.

Manual follow-ups: none.

### v0.3.0 (2026-02-24)

Automatic migrations:

* Schema v2 introduces MCP proxy history preprocessing keys and normalizes allowed origins. `migrateIfNeeded` stamps the version; no data is lost.

## Settings Schema Migrations

| From | To | What changes |
| :--- | :--- | :--- |
| v1 (or unversioned) | v2 | Normalize `mcp.allowedOrigins` into a clean list; replace the legacy Gemini default command with the current `--output-format text --model gemini-2.5-flash --yolo` form. |
| v2 | v3 | Stamp without data movement; new `custom.prompt.library.v1` key is loaded on demand (malformed JSON yields an empty library). |

Migrations are additive and run at load time inside `AgentSettings.migrateIfNeeded`. Downgrades are not supported — delete `preferences()` and start fresh if you need to run an older version.

See [Settings Migration](../developer/settings-migration.md) for the developer-facing reference.

## After the Upgrade

1. Reload the extension (`Extensions → Installed → ...` → remove and re-add the JAR).
2. Open the **AI Agent** tab and verify:
   * The Preferred Backend still resolves (especially relevant if you used `burp-ai` and forgot to enable *Use AI for extensions*).
   * The **Privacy Mode** indicator matches your intent.
   * MCP status (if you use it) is `Running` with the expected port.
3. Trigger one context-menu action and confirm the [Context Preview Dialog](../privacy/privacy-modes.md) shows the expected redacted payload.
4. If you use audit logs, tail `~/.burp-ai-agent/audit.jsonl` for a new `prompt` event and verify the `payloadSha256` field is present.

## Related Pages

* [Changelog](changelog.md)
* [Naming & Paths](naming-and-paths.md)
* [Configuration Directory](configuration-directory.md)
* [Settings Migration (Developer)](../developer/settings-migration.md)
