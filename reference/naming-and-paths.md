# Naming & Paths

The extension is called **Custom AI Agent**, but several internal identifiers still use the legacy name `burp-ai-agent`. This page explains why and lists exactly where to expect each form.

## Product vs Identifier Reference

| Context | Value | Notes |
| :--- | :--- | :--- |
| Product and Burp extension name | `Custom AI Agent` | Visible in Burp's Extensions list and in the main tab. |
| Release artifact | `Custom-AI-Agent-<version>.jar` | Since v0.6.0. Older releases shipped as `Burp-AI-Agent-*.jar`. |
| Checksum / SBOM | `Custom-AI-Agent-<version>.jar.sha256`, `bom.json` | Attached to every GitHub release since v0.6.0. |
| Burp tab title | `AI Agent` | Short form; does not include the word "Custom". |
| GitHub repository | `github.com/six2dez/burp-ai-agent` | Repo URL kept to avoid breaking external links, issues, and forks. |
| Runtime directory | `~/.burp-ai-agent/` (Windows: `%USERPROFILE%\.burp-ai-agent\`) | Kept to preserve upgrades; see [Configuration Directory](configuration-directory.md). |
| Java package | `com.six2dez.burp.aiagent` | Kept to preserve backend SPI compatibility for drop-in JARs. |
| Gradle project name | `burp-ai-agent` (in `settings.gradle.kts`) | Internal only. |
| MCP implementation string | `burp-ai-agent` | Shows up in MCP `initialize` responses; arbitrary and clients may rename. |
| MCP server name in client configs (examples) | `burp-ai-agent` | Suggested key in `claude_desktop_config.json` and similar; you can call it anything you like. |
| Audit log path | `~/.burp-ai-agent/audit.jsonl` | Same directory as above. |

## Why the Two Names Coexist

The rebranding from "Burp AI Agent" to "Custom AI Agent" happened in **v0.5.0** (see [Changelog](changelog.md#0-5-0-2026-04-02)) as part of a PortSwigger naming-guidance update: third-party extensions should not use "Burp" as the leading word in a product name. Only user-facing strings and the JAR artifact were changed. Internal identifiers stayed the same on purpose, for three reasons:

1. **Migrations**: existing users already have `~/.burp-ai-agent/` populated with audit logs, prompt caches, agent profiles, and drop-in backend JARs. Renaming the directory would invalidate all of that silently.
2. **SPI compatibility**: external drop-in backends ship JARs that register under `com.six2dez.burp.aiagent.backends.AiBackendFactory` via `META-INF/services`. Renaming the package would break every published third-party backend.
3. **GitHub identifiers**: repo URL, issue IDs, and pull requests are stable references that appear in pinned dependencies, automation, and user bookmarks.

## What to Expect in Each Surface

* **UI, chat labels, audit payloads, documentation titles** → `Custom AI Agent`.
* **Filesystem paths, repo URL, Java imports, Gradle tasks** → `burp-ai-agent`.
* **MCP client configs** → either is fine; the identifier is advisory. Prefer `burp-ai-agent` if you want copy-paste examples from the docs to keep working.

## Ops and CI Migration Checklist

If your pipelines pre-date v0.5.0, update the following:

* Scripts that hardcode `Burp-AI-Agent-*.jar` (download URLs, rsync globs, symlinks) must be renamed to `Custom-AI-Agent-*.jar`.
* Release notifications or changelog scrapers that match `Burp AI Agent` in text should now match `Custom AI Agent`.
* Dashboards or Slack messages that reference the tab name `Burp AI Agent` should use `AI Agent` (the tab was shortened at the same time).

Scripts that reference `~/.burp-ai-agent/`, the GitHub repo URL, the Java package, or the MCP implementation string do **not** need to change.

## Related Pages

* [Configuration Directory](configuration-directory.md)
* [Upgrade & Migration Guide](upgrade-migration.md)
* [Changelog](changelog.md)
