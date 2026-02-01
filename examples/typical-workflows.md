# Typical Workflows

These workflows demonstrate how to use the Burp AI Agent effectively in real-world security assessments.

## Bug Bounty Triage

Rapidly assess endpoints and generate reports for submission.

1.  Browse the target application with Burp Proxy.
2.  In **Proxy → HTTP History**, right-click an interesting request → **Find vulnerabilities**.
3.  Review the AI's analysis in the chat panel.
4.  If a vulnerability is identified, right-click the same request → **AI Active Scan** (with `SAFE` risk level).
5.  If confirmed, use the chat to ask: *"Generate a PoC with curl commands for this finding."*
6.  Create an issue in Burp via the MCP `issue_create` tool or manually.
7.  Use **Full report** to generate a structured write-up for submission.

## Large Scope Reconnaissance

Efficiently map and analyze a large application surface.

1.  Set your target in **Target → Scope**.
2.  Enable the **Passive** toggle in the top bar.
3.  Browse the application thoroughly (or use Burp's crawler on Pro).
4.  The passive scanner automatically analyzes traffic in the background.
5.  Check findings in the extension's **View Findings** panel.
6.  Filter by severity (HIGH/CRITICAL) and review the most interesting endpoints.
7.  For promising findings, right-click the request → **Find vulnerabilities** for a deeper analysis.
8.  Promote high-confidence findings by enabling **Auto-Queue to Active**.

## API Security Assessment

Systematic testing of REST/GraphQL APIs.

1.  Proxy API traffic through Burp.
2.  Right-click API endpoints → **Analyze this request** to understand each endpoint's purpose, parameters, and auth mechanism.
3.  For authentication endpoints, use **Login sequence** to document the auth flow.
4.  Test authorization with **Access control** to generate a test plan for IDOR/BOLA/BAC.
5.  Enable the passive scanner with **Scope Only** to catch common API misconfigurations.
6.  Use the MCP server with Claude Desktop for agentic testing: *"Check all API endpoints in proxy history for missing authorization checks."*

## Agentic Pentesting with MCP

Let an external AI agent drive Burp autonomously under your supervision.

1.  Enable the **MCP** toggle. Note the token from **MCP Server tab in the bottom settings panel**.
2.  Configure Claude Desktop (or another MCP client) with the Burp MCP server connection.
3.  Start a conversation: *"List the last 20 requests in proxy history for the target domain."*
4.  The AI calls `proxy_http_history_regex` and returns results.
5.  Ask: *"Analyze the `/api/users/{id}` endpoint for IDOR. Send test requests with different IDs."*
6.  The AI calls `http1_request` to send test payloads and reports differences.
7.  If a vulnerability is found: *"Create an issue in Burp with the evidence."*
8.  The AI calls `issue_create` with full details.

> **Safety note**: Enable unsafe MCP tools only when you are actively supervising the AI agent. Disable them when not in use.

## JavaScript Analysis

Deep-dive into client-side code for security issues.

1.  Browse the target and let Burp capture JavaScript responses.
2.  In **Proxy → HTTP History**, find JS files.
3.  Right-click → **Explain JS** to get a summary of the code's behavior and security implications.
4.  For large JavaScript bundles, use **Gemini** as the backend (1M+ token context window).
5.  Ask follow-up questions in the chat: *"Are there any hardcoded API keys or secrets in this JavaScript?"*

## Compliance Audit with Audit Logging

Produce a verifiable record of all AI interactions for compliance.

1.  Enable **Audit Logging** in **Privacy & Logging tab in the bottom settings panel**.
2.  Set **Privacy Mode** to **STRICT** for sensitive engagements.
3.  Enable **Determinism Mode** for reproducible prompts.
4.  Perform your assessment normally using context menus and chat.
5.  After the assessment, review `~/.burp-ai-agent/audit.jsonl` for the complete interaction log.
6.  Use the SHA-256 hashes in the audit log to verify that context data hasn't been modified.
7.  Export prompt bundles via the audit logger's ZIP export for archival.
