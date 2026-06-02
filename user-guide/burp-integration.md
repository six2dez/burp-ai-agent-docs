# Burp Integration

The extension integrates directly with Burp tools and workflows so AI analysis stays close to real testing data.

## Supported Burp Tools

| Burp Tool | Integration |
| :--- | :--- |
| **Proxy History** | Request context menus and passive scan coverage (via a registered Montoya passive scan check, Pro). |
| **Repeater** | Context actions on requests/responses and MCP Repeater tools. |
| **Intruder** | MCP tools can create and run Intruder setups. |
| **Scanner** (Pro) | Issue context menus, active checks, and ScanCheck integration. |
| **Site Map** | Context menus and MCP site map search/query tools. |
| **Target Scope** | Scope-aware scanner and MCP scope tools. |
| **Comparer** (Pro) | MCP `comparer_send` workflow support. |
| **Collaborator** (Pro) | Active scanner OAST payload generation and polling. |

## Burp Pro vs Community Edition

| Feature | Community | Professional |
| :--- | :--- | :--- |
| Context menu actions (requests) | Yes | Yes |
| Context menu actions (issues) | No | Yes |
| Chat & sessions | Yes | Yes |
| All AI backends | Yes | Yes |
| MCP server | Yes (non-Pro tools) | Yes (all tools) |
| Passive AI Scanner | Manual queue path | Automatic passive scan check + manual queue |
| Active AI Scanner | Manual queue path | Native scanner integration + queue |
| Scanner MCP tools | No | Yes |
| Collaborator OAST | No | Yes |
| Scan reports via MCP | No | Yes |

The extension detects Burp edition at startup and disables unsupported capabilities automatically.

## MCP Tool Toggles

You control MCP exposure from **Burp Integration** and **MCP Server** tabs. The **Burp Integration** tab embeds the redesigned MCP Tools panel: tools are grouped **extension-native (AI)** vs **generic (Montoya)**, each tagged **store-build** / **full-build**, with a search/filter box and per-group bulk toggles.

### Build matters

There are two build artifacts, and which tools are even registered depends on the build:

* **Full build** (default, GitHub releases — `Custom-AI-Agent-full-0.8.0.jar`): registers all **59** MCP tools, including the generic Montoya-API tools (proxy history, Repeater, scanner, scope, site map, Intruder, Collaborator, utilities, etc.).
* **Store build** (BApp Store — `Custom-AI-Agent-0.8.0.jar`): registers only the **8** extension-native AI tools (`status`, `issue_create`, `ai_analyze`, `ai_passive_scan`, `ai_findings_recent`, `redact_preview`, `ai_audit_query`, `ai_backends_list`). The generic Montoya-API tools are not exposed over MCP in this build — PortSwigger's official Burp MCP Server provides those.

The tags in the panel show which group each tool belongs to so you can see at a glance what is available in your build.

### Safe vs Unsafe

* **Safe**: read-only operations, enabled by default.
* **Unsafe**: state-changing or traffic-generating operations, disabled by default.

### Managing Tool Access

1. Open the **Burp Integration** tab in Settings.
2. Enable/disable tools by group, using search/filter and per-group bulk toggles as needed.
3. Enable **Unsafe Tools** in the **MCP Server** tab if unsafe tool toggles must be active.

![Screenshot: MCP tool toggles](../.gitbook/assets/mcp-tool-toggles.png)

{% hint style="warning" %}
Enable unsafe tools only for trusted MCP clients and only while actively using those workflows.
{% endhint %}

## Collaborator Workflow (Pro)

When **Use Collaborator (OAST)** is enabled in Active Scanner settings, the workflow is:

1. Active scanner builds a targeted OAST payload using a Burp Collaborator interaction domain.
2. Payload is inserted into selected injection points (based on risk level and scan mode).
3. Requests are sent to the target and polling runs at configured intervals.
4. DNS/HTTP callbacks are correlated to the originating scan item.
5. Confirmed OAST behavior contributes to scanner evidence and issue creation.

Typical use cases:

* blind SSRF confirmation,
* blind command injection confirmation,
* out-of-band deserialization indicators.

## Native Scanner Integration (Pro)

On Burp Pro, an `AiScanCheck` is registered into Burp's scanner pipeline.

* AI checks run alongside native checks.
* Findings appear as Burp issues (`[AI Active]` naming convention).
* Burp scope/configuration rules still apply.

On Community edition, this path is skipped and the extension uses manual queue execution.

## Related Pages

* [Active AI Scanner](../scanners/active.md)
* [Context Menus](context-menus.md)
* [MCP Security Model](../mcp/security-model.md)
