# Glossary

*   **MCP (Model Context Protocol)**: An open standard that enables AI models to interact with local tools and data sources securely. The Burp AI Agent implements an MCP server that exposes Burp functionality to external AI agents.

*   **SSE (Server-Sent Events)**: A streaming method used by the MCP server to push real-time updates and tool results to the client over HTTP.

*   **STDIO Bridge**: An alternative MCP transport that uses standard input/output (stdin/stdout) instead of HTTP/SSE. Useful for process-based MCP clients.

*   **Deterministic Redaction**: A privacy technique ensuring that a specific piece of sensitive data (like a hostname) is always replaced by the same pseudonym throughout a session, allowing the AI to maintain logical consistency across multiple requests.

*   **Redaction Salt**: A secret string used to generate stable pseudonyms for hostnames in STRICT mode. Should be rotated between engagements.

*   **Backend Supervisor**: The internal component that manages the lifecycle (start, stop, monitor, auto-restart) of AI CLI processes and HTTP backend connections.

*   **MCP Supervisor**: A separate component managing the MCP server lifecycle, including health checks, restart logic, and transport management.

*   **In-Scope Filter**: A setting that restricts AI analysis only to targets explicitly added to the Burp Suite Target Scope.

*   **Prompt Bundle**: The complete package sent to the AI: prompt template + redacted context + metadata. Hashed with SHA-256 for audit integrity.

*   **Prompt Template**: The default instruction text associated with each context menu action (e.g., "Find vulnerabilities", "Generate PoC"). Customizable in settings.

*   **Backend Adapter**: An implementation that connects the extension to a specific AI provider (Ollama, Claude, Gemini, etc.). Follows the factory pattern for pluggability.

*   **ServiceLoader**: Java's built-in plugin discovery mechanism, used to register backend adapters (both built-in and external JAR drop-ins).

*   **Tool Gating**: The security mechanism that categorizes MCP tools as "safe" (read-only, enabled by default) or "unsafe" (state-modifying, disabled by default).

*   **ScanCheck**: A Burp Pro API interface that allows extensions to register custom scan checks with the native scanner. The AI scanner uses this for integration.

*   **OAST (Out-of-Band Application Security Testing)**: A technique where payloads trigger external interactions (DNS/HTTP callbacks) to detect blind vulnerabilities. Uses Burp Collaborator.

*   **VulnContext**: Internal model tracking contextual information about a vulnerability — whether it affects auth endpoints, payment functionality, admin areas, or contains PII — for severity adjustment.

*   **Injection Point**: A location in an HTTP request where user input can be modified for testing (URL parameter, body parameter, header, cookie, JSON field, XML element, path segment).

*   **Detection Method**: The technique used to determine if a payload successfully exploited a vulnerability (error-based, blind boolean, blind time, reflection, out-of-band, content-based).

*   **JSONL (JSON Lines)**: The audit log format where each line is a standalone JSON object. Easy to parse with `jq`, import into SIEMs, or process with scripts.

*   **Scan Mode**: The vulnerability class selection strategy for the active scanner (BUG_BOUNTY, PENTEST, FULL), determining which vulnerability types are tested.

*   **Drop-in Backend**: A custom backend packaged as a JAR file and placed in `~/.burp-ai-agent/backends/` for automatic discovery without modifying the extension.

*   **Privacy Mode**: One of three data protection levels (STRICT, BALANCED, OFF) controlling what information is redacted before being sent to AI backends.

*   **Risk Level**: The payload danger classification for the active scanner (SAFE, MODERATE, DANGEROUS), controlling what types of test payloads are allowed.

*   **Agent Profile**: A Markdown file in `~/.burp-ai-agent/AGENTS/` that provides role-specific system instructions to the AI. Profiles use `[SECTION_NAME]` headers to map instructions to specific context menu actions.
