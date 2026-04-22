# Burp Scan Skill

The **Burp Scan Skill** is a standalone knowledge file that lets you use any AI coding assistant (Claude Code, Gemini CLI, Codex, OpenCode, Copilot, etc.) as a Burp scanner from your terminal. Instead of relying on the plugin's built-in AI backends, your preferred terminal AI becomes the reasoning engine while Burp provides 53+ tools via MCP.

## Why Use the Skill?

The plugin has two layers that serve different purposes:

| Plugin (Built-in Scanner) | Skill (Terminal AI) |
| :--- | :--- |
| Automated background scanning | Interactive, analyst-guided scanning |
| Real-time proxy hooks (intercepts every response) | Pulls proxy history on demand |
| JVM-native performance (regex at JVM speed) | MCP tool calls over network (slower but smarter) |
| Works with any backend (Ollama, LM Studio, etc.) | Uses your terminal AI (Claude Code, Gemini, etc.) |
| Set-and-forget | Human-in-the-loop |

**They are complementary**, not replacements for each other. The plugin scans automatically in the background, while the skill lets you run interactive, targeted scans with the full reasoning power of your preferred AI model.

### When to Use the Skill

* You prefer working in your terminal with Claude Code, Gemini CLI, or another AI assistant
* You want analyst-guided scanning where you control what gets tested
* You want to combine Burp's tools with your AI's broader knowledge
* You need to run targeted tests on specific endpoints or vulnerability classes
* You want to chain Burp scanning with other tools (recon, code review, report writing)

## Installation

The skill file is located at `skills/burp-scan/SKILL.md` in the [repository](https://github.com/six2dez/burp-ai-agent).

### Claude Code

Copy the skill to your Claude Code skills directory:

{% hint style="info" %}
The GitHub repository is still called `burp-ai-agent` even though the extension is now published as **Custom AI Agent** — see [Naming & Paths](../reference/naming-and-paths.md).
{% endhint %}

```bash
# Clone the repo (if you haven't already)
git clone https://github.com/six2dez/burp-ai-agent.git

# Global installation (available in all projects)
cp -r burp-ai-agent/skills/burp-scan ~/.claude/skills/burp-scan

# Or project-specific installation
mkdir -p .claude/skills
cp -r burp-ai-agent/skills/burp-scan .claude/skills/burp-scan
```

Once installed, you can invoke it with `/burp-scan` or let it trigger automatically when you mention Burp scanning.

### Gemini CLI

Add the skill as a context file:

```bash
# Copy to your Gemini configuration
cp burp-ai-agent/skills/burp-scan/SKILL.md ~/.gemini/context/burp-scan.md
```

Or paste the content into your system prompt configuration.

### Codex / OpenCode / Copilot

These assistants support system prompts or context files. Include the `SKILL.md` content as system context alongside your MCP connection configuration.

### Any Other LLM / AI Assistant

The skill is a self-contained Markdown file. You can use it with any AI that accepts context:

1. Download `skills/burp-scan/SKILL.md` from the repository
2. Feed it as context / system prompt to your AI
3. Configure your AI to connect to Burp's MCP server
4. Start scanning

## Prerequisites

Before using the skill, ensure:

1. **Burp Suite is running** (Community or Professional)
2. **The AI Agent extension is loaded** and active
3. **MCP server is enabled** in Settings > MCP Server
4. **Your AI assistant is connected** to the MCP server (via SSE, stdio, or HTTP)

### MCP Connection Setup

The skill communicates with Burp through MCP tools. Your AI assistant needs to be connected to Burp's MCP server.

**For Claude Code** (using supergateway):

Add to your MCP configuration:

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

**For other clients**: Connect to `http://127.0.0.1:9876/sse` (default MCP endpoint). If External Access is enabled, include `Authorization: Bearer <token>` in requests.

## What the Skill Contains

The skill is organized into 6 sections:

### 1. MCP Tool Reference Card

All 53+ MCP tools reorganized by scanning action (not by Burp UI category):

* **Discover scope & surface**: `scope_check`, `site_map`, `proxy_http_history`, `response_body_search`
* **Analyze traffic**: `params_extract`, `find_reflected`, `insertion_points`, `request_parse`, `diff_requests`
* **Send test payloads**: `http1_request`, `http2_request`, `repeater_tab`, `intruder`
* **OOB verification**: `collaborator_generate`, `collaborator_poll`
* **Encoding & utility**: `url_encode/decode`, `base64_encode/decode`, `jwt_decode`, `hash_compute`
* **Report findings**: `issue_create`, `scanner_issues`
* **Control Burp scanner**: `scan_audit_start`, `scan_crawl_start`, `scan_task_status`

### 2. Passive Analysis Protocol

A step-by-step protocol for analyzing proxy traffic without sending additional requests:

1. **Pull traffic** from proxy history (filtered by scope, excluding static assets)
2. **Local pattern checks** (deterministic, no AI needed):
   * Request smuggling indicators (CL+TE, duplicate CL)
   * CSRF absence (state-changing + cookie auth, no token)
   * Deserialization surface (Java serialized markers)
   * Unrestricted file upload (dangerous extensions accepted)
3. **Context extraction**: Headers, parameters, auth mechanisms, potential object IDs, tech stack hints
4. **Analysis checklist**: Injection, auth/access control, info disclosure, configuration, high-value targets, API security
5. **Severity definitions** and **exclusion rules** (what NOT to report)
6. **JS endpoint discovery**: Extract hidden API routes from JavaScript files

### 3. Active Testing Payload Library

200+ payloads organized by vulnerability class with detection methods and expected evidence:

| Vuln Class | Payloads | Detection Method |
| :--- | :--- | :--- |
| **SQL Injection** | Error-based (`'`, `"`, `';--`), Blind boolean (`AND 1=1`/`AND 1=2`), Time-based (`SLEEP(5)`, `pg_sleep(5)`, `WAITFOR DELAY`), UNION-based | Error patterns (95%), response diff, 5s delay |
| **XSS Reflected** | 8 payloads with unique marker `XSS-BURP-AI-1337` | Marker reflection in response (95%) |
| **LFI / Path Traversal** | 10 variants (traversal, encoding, null byte, extension bypass) | `root:x:0:0` or `[fonts]` in response (95%) |
| **SSTI** | 10 template syntaxes (`{{1337*73}}`, `${1337*73}`, `<%= 1337*73 %>`) | Math result `97601` in response (95%) |
| **Command Injection** | 11 variants (`; id`, `\| id`, `` `id` ``, `$(id)`, blind sleep) | `uid=XXX(user) gid=XXX` pattern (95%) |
| **SSRF** | 10 targets (localhost variants, cloud metadata, protocol schemes) | Local service response, metadata content |
| **XXE** | 2 payloads (Linux + Windows file read) | File content in response (95%) |
| **IDOR** | Context-aware: ID-1, ID+1, ID=1, ID=0, ID=-1, UUID mutation | Different user data for manipulated ID |
| **Host Header Injection** | Marker `evil-burp-ai-test.com` | Marker reflected in body or Location header |
| **OAuth** | 4 redirect\_uri bypass patterns | Redirect accepted to attacker domain |
| **CORS** | Origin manipulation (`evil.com`, `null`) | ACAO header reflects attacker origin |
| **Open Redirect** | 5 bypass patterns (`//evil.com`, `/\evil.com`) | Redirect to evil domain |
| **Cache Poisoning** | X-Forwarded-Host injection | Marker in cached response |
| **Git/Backup Exposure** | `.git/HEAD`, `.git/config`, `.svn/entries`, backup extensions | Version control content patterns |
| **Debug Endpoints** | `/actuator`, `/_profiler`, `/telescope`, `/phpinfo.php` | Debug information in response |
| **Price Manipulation** | Negative, zero, near-zero, overflow values | Value accepted without validation |

The skill also includes a **context-aware payload generation** template for creating technology-specific payloads when you know the target's tech stack.

**Safety**: The skill explicitly blocks destructive payloads (DROP, DELETE, TRUNCATE, ALTER, SHUTDOWN, rm, etc.).

### 4. Scanning Workflow Protocol

An end-to-end decision tree for running a complete scan:

```
Phase 1: Scope & Reconnaissance
  scope_check -> site_map -> proxy_http_history -> identify tech stack & auth

Phase 2: Passive Analysis
  For each request: local checks -> context extraction -> analysis checklist
  JS endpoint discovery -> test discovered endpoints

Phase 3: Active Confirmation
  For each passive finding: select payloads -> send via http1_request -> analyze response
  Detection methods: ERROR_BASED, REFLECTION, CONTENT_BASED, BLIND_BOOLEAN, BLIND_TIME, OUT_OF_BAND

Phase 4: OOB Testing
  collaborator_generate -> inject payload -> wait -> collaborator_poll

Phase 5: Knowledge Tracking
  Track per-host: tech stack, auth info, error patterns, prior findings
  Use knowledge to prioritize and generate adaptive payloads
```

### 5. Issue Creation Protocol

Standardized format for reporting confirmed vulnerabilities through `issue_create`:

* **Name format**: `[Vuln Type] - [Specific Detail]`
* **Severity mapping**: All 62 vuln classes mapped to HIGH/MEDIUM/LOW/INFORMATION
* **Confidence mapping**: CERTAIN (>=95%), FIRM (>=85%), TENTATIVE (>=70%)
* **Remediation reference**: Mitigation text for every vulnerability class

### 6. Vulnerability Classes Reference

Complete reference of all 62 vulnerability classes organized by OWASP Top 10:

* Scan modes: BUG\_BOUNTY (high-impact only), PENTEST (exhaustive), FULL (all classes)
* Passive-only vs active-testable classification
* Impact context multipliers (auth, payment, admin, PII, API endpoints)
* False positive indicators for each major vuln class

## Usage Examples

### Full Scan

```
You: I have Burp running with proxy traffic from target.com.
     Run a full passive scan and confirm any findings with active testing.

AI: [Checks scope] [Pulls proxy history] [Analyzes each request/response]
    [Finds potential SQLi in /api/users?id=123]
    [Sends error-based payloads via http1_request]
    [Confirms SQLi: "You have an error in your SQL syntax" in response]
    [Creates Burp audit issue via issue_create]
```

### Targeted IDOR Scan

```
You: Check the /api/v2/accounts endpoint for IDOR.
     My user ID is 4521.

AI: [Sends http1_request with id=4520, id=4522, id=1, id=0]
    [Compares responses - id=4520 returns different user data]
    [Confirms IDOR: user data for another account returned]
    [Creates HIGH severity issue with evidence]
```

### OOB Blind Testing

```
You: Test /api/import for XXE. The endpoint accepts XML.

AI: [Generates Collaborator payload via collaborator_generate]
    [Sends XXE payload with Collaborator domain]
    [Waits, then runs collaborator_poll]
    [DNS interaction detected -> XXE confirmed]
    [Creates CERTAIN confidence issue]
```

### JS Endpoint Discovery

```
You: Find hidden API endpoints in the JavaScript files from proxy history.

AI: [Searches proxy_http_history_regex for .js files]
    [Extracts API paths: /api/v1/admin/users, /api/internal/debug]
    [Tests each endpoint for unauthorized access]
    [Finds /api/internal/debug returns 200 without auth]
    [Creates issue for exposed debug endpoint]
```

## Relationship to Other Skills

If you use Claude Code, the `burp-scan` skill integrates with the broader security skill ecosystem:

| Skill | Relationship |
| :--- | :--- |
| `bug-bounty` | Orchestrator. Calls `burp-scan` when it's time to test via Burp. |
| `security-arsenal` | Generic payload library. Complements `burp-scan`'s Burp-specific payloads. |
| `web2-vuln-classes` | Deep vulnerability theory. Reference when you need technique details. |
| `h1-brain` | Real-world attack patterns from disclosed HackerOne reports. |
| `triage-validation` | Post-scan validation. Decides if findings are submission-worthy. |
| `report-writing` | Report generation. Produces submission-ready reports from findings. |

## Comparison with Plugin's Built-in Scanners

| Feature | Plugin Passive Scanner | Plugin Active Scanner | Burp Scan Skill |
| :--- | :--- | :--- | :--- |
| **Trigger** | Automatic (proxy hook) | Auto from passive or manual queue | Manual (user initiates) |
| **AI Model** | Configured backend (Ollama, OpenAI, etc.) | Same | Your terminal AI (Claude, Gemini, etc.) |
| **Speed** | Fast (JVM-native regex) | Fast (concurrent threads) | Slower (MCP round-trips) |
| **Intelligence** | Limited by prompt template | Static + adaptive payloads | Full AI reasoning |
| **Context** | Single request/response | Injection point focused | Entire scan history |
| **Customization** | Prompt templates | Scan mode + risk level | Fully interactive |
| **Requires** | Backend configured in plugin | Plugin scanner enabled | MCP + terminal AI |
| **Best for** | Background monitoring | Automated confirmation | Deep manual testing |

## Troubleshooting

### "Tool not found" errors

Verify the MCP connection is working:

1. Check that MCP is enabled in Settings > MCP Server
2. Test the connection: the `status` tool should return Burp version info
3. If using supergateway, ensure Node.js 18+ is installed

### "Unsafe tool blocked" errors

Active testing tools (`http1_request`, `http2_request`, `repeater_tab`, `intruder`) require **Unsafe Mode** enabled in Settings > MCP Server. Enable it and optionally configure which unsafe tools to allow.

### Slow response times

MCP tool calls go over the network. For faster scanning:

* Use the plugin's built-in scanner for bulk automated testing
* Use the skill for targeted, high-value manual testing
* Reduce the number of payloads per injection point

### No proxy history

Make sure you have browsed the target through Burp Proxy and that traffic appears in Proxy > HTTP History before running the skill.
