# Which Build?

Custom AI Agent ships as two builds from one repository, the BApp Store build and the GitHub full build: pick one and load only that one.

## The two builds

| | BApp Store build | GitHub full build |
| :--- | :--- | :--- |
| Gradle task | `./gradlew bappJar` | `./gradlew fullJar` |
| Output file | `build/libs/custom-ai-agent-bapp.jar` | `build/libs/Custom-AI-Agent-full-<version>.jar` |
| Release asset | `Custom-AI-Agent-bapp-<version>.jar` and `Custom-AI-Agent-bapp-<version>.jar.sha256` | `Custom-AI-Agent-full-<version>.jar` and `Custom-AI-Agent-full-<version>.jar.sha256` |
| Where to get it | [GitHub Releases](https://github.com/six2dez/burp-ai-agent/releases) today; the BApp Store once PortSwigger lists it (see below) | [GitHub Releases](https://github.com/six2dez/burp-ai-agent/releases) |
| MCP tools | 8 extension-native tools: `status`, `issue_create`, `ai_analyze`, `ai_passive_scan`, `ai_findings_recent`, `redact_preview`, `ai_audit_query`, `ai_backends_list` | All 59: the same 8 plus 51 generic Montoya API wrappers |
| AI backends | 7: Burp AI, Anthropic, Ollama, LM Studio, NVIDIA NIM, Perplexity, Generic OpenAI-compatible | 12: the same 7 plus Gemini CLI, Claude CLI, Codex CLI, OpenCode CLI, Copilot CLI |
| External MCP servers | Not included | Included: [External MCP Servers](../mcp/external-servers.md) and their `ext:<server>:<tool>` tools |
| Passive-to-active | Suggested targets, queued by hand ("Queue N suggested targets" in the AI Active Scanner Queue dialog) | The same, plus the opt-in "Auto-queue passive findings for active testing", off by default |
| Burp "Use AI" | Needed by every AI backend, Burp AI included, and by MCP `ai_analyze` and `ai_passive_scan` | Needed by Burp AI and by MCP `ai_analyze` and `ai_passive_scan`; the HTTP and CLI backends do not depend on it |
| Health checks | On demand only: Save, a backend change, a click on the status pill, **Test connection** or the first chat message. The pill reads "Not checked · click to test" until then | A startup check for every backend, then every 30 s for local backends (Burp AI, the CLI backends, HTTP servers on a loopback address); remote providers again after a settings or backend change and on a pill click |
| Drop-in backend JARs | Not supported: the build never creates, lists or loads `~/.burp-ai-agent/backends` | Loaded from `~/.burp-ai-agent/backends` ([Adding a Backend](../developer/adding-backend.md)) |
| AI scanners at load | OFF at every load; switching one on lasts until the extension unloads | The same |
| Name in Burp | **Custom AI Agent**; the Output tab banner reads `Custom AI Agent <version> (BApp Store build)` | **Custom AI Agent**; the Output tab banner reads `Custom AI Agent <version> (GitHub full build)` |

The **Settings > Help** caption names the running build the same way. In **Settings > MCP Server**, the native tools are listed under "Extension-native tools — available in both the BApp Store and the full build." and the generic tools under "Generic Montoya API wrappers — available in the full build only."

## Burp edition requirements

The BApp Store build runs on Burp Community and Burp Professional; it sends AI requests only while Burp's "Use AI" setting is enabled for the extension, and with it off it loads and refuses every AI request.

The BApp Store build checks Burp's "Use AI" setting through api.ai().isEnabled() at each AI request. While it is off, the extension still loads and its MCP server, native tools and privacy settings keep working, but every AI request is refused and the status pill shows "AI disabled in Burp". Which Burp editions can turn "Use AI" on is decided by Burp, not by this extension, and it was not measured on Burp Community. For local or CLI AI that does not depend on "Use AI", use the GitHub full build: its HTTP and CLI backends do not depend on that setting, while Burp AI and the MCP ai_* tools still do.

What each build does with "Use AI" on and off. This is the gate the code enforces; it is not a statement about any Burp edition:

| "Use AI" for the extension | BApp Store build | GitHub full build |
| :--- | :--- | :--- |
| On | AI requests go to the selected backend | AI requests go to the selected backend |
| Off | Loads. The MCP server, the native tools and the privacy settings keep working. Every AI request is refused: the chat shows the refusal below, the status pill reads "AI disabled in Burp" and the scanner panel reads "Paused: Use AI is off" | Loads. Burp AI is refused; the HTTP and CLI backends keep working. MCP `ai_analyze` and `ai_passive_scan` return a tool error with the reason |

The BApp Store build's refusal in the chat reads: "This build sends AI requests only while Burp's 'Use AI' setting is enabled. Turn it on in Burp's settings to use any AI backend."

**Where "Use AI" lives.** PortSwigger documents "Use AI" as a per-extension checkbox, off by default: **Extensions > Installed > Custom AI Agent > Use AI**.

## BApp Store or GitHub Releases

Custom AI Agent is not listed in the BApp Store yet: the BApp Store build is going through PortSwigger's review in [extension-portal#231](https://github.com/PortSwigger/extension-portal/issues/231).

From v1.1.0, every [GitHub release](https://github.com/six2dez/burp-ai-agent/releases) attaches both JARs, `Custom-AI-Agent-bapp-<version>.jar` and `Custom-AI-Agent-full-<version>.jar`, each with a `.sha256` file in `sha256sum` format, plus one `bom.json` (CycloneDX SBOM). Check a download with `sha256sum -c <asset>.sha256` in the folder that holds both files. The checksum is published on the same release as the JAR, so it shows the file is complete, not who built it. `bom.json` lists the full build's runtime dependencies, which include every dependency of the BApp Store build.

## Load only one build

{% hint style="warning" %}
**Load only one build.** The full build and the BApp Store build both register as **Custom AI Agent**, so they look identical in Burp's Extensions list. Loading both at once gives you two Custom AI Agent tabs, two MCP servers competing for the same port, and two copies writing the shared `~/.burp-ai-agent` directory. Unload one before you load the other.
{% endhint %}

Both builds share Burp's saved settings and encrypted secrets, so moving from one to the other loses nothing. A BApp Store build loaded over a saved CLI backend runs on Burp AI and shows a notice; the saved choice is kept for the full build.

## Building from source

Both builds need JDK 21:

```bash
git clone https://github.com/six2dez/burp-ai-agent.git
cd burp-ai-agent

# BApp Store build -> build/libs/custom-ai-agent-bapp.jar
./gradlew clean bappJar

# GitHub full build -> build/libs/Custom-AI-Agent-full-<version>.jar
./gradlew clean fullJar
```

The old single-JAR Gradle task is retired: running it fails with a message naming the two tasks above. See [Installation](installation.md) for loading the JAR into Burp.
