# Glossary

* **MCP (Model Context Protocol)**: Open protocol used by AI clients to invoke local tools safely.
* **SSE (Server-Sent Events)**: Streaming transport used by MCP HTTP sessions.
* **STDIO Bridge**: MCP transport over stdin/stdout for process-based clients.
* **Deterministic Redaction**: Stable replacement of sensitive values so identical inputs map to identical pseudonyms.
* **Redaction Salt**: Secret used to produce stable host pseudonyms in STRICT mode.
* **Backend Supervisor**: Component that manages backend process lifecycle and restart behavior.
* **MCP Supervisor**: Component that manages MCP server lifecycle and health behavior.
* **In-Scope Filter**: Setting that restricts analysis to Burp in-scope targets.
* **Prompt Bundle**: Prompt template plus redacted context and metadata, hashed for audit traceability.
* **Prompt Template**: Default instruction text used by context menu actions.
* **BountyPrompt Action**: Curated request/response context action loaded from BountyPrompt JSON prompts.
* **Tag Resolver**: Component that replaces BountyPrompt `[HTTP_*]` tags with selected, redacted context fields.
* **Confidence Gate**: Threshold check used before auto-creating BountyPrompt issues.
* **Backend Adapter**: Provider-specific implementation for CLI or HTTP model backends.
* **ServiceLoader**: Java plugin mechanism used for backend adapter discovery.
* **Tool Gating**: Safety mechanism that controls safe/unsafe MCP tool availability.
* **ScanCheck**: Burp Pro interface used by custom scanner checks.
* **OAST (Out-of-Band Application Security Testing)**: Detection technique using external callbacks (for example Burp Collaborator).
* **VulnContext**: Internal model storing vulnerability context signals used in prioritization.
* **Injection Point**: Modifiable request location for active payload testing.
* **Detection Method**: Evidence type used to confirm findings (error, blind, reflection, OOB, and similar).
* **JSONL (JSON Lines)**: Log format where each line is an independent JSON object.
* **Scan Mode**: Active scanner class-selection strategy (`BUG_BOUNTY`, `PENTEST`, `FULL`).
* **Drop-in Backend**: External backend JAR loaded from `~/.burp-ai-agent/backends/`.
* **Privacy Mode**: Redaction policy level (`STRICT`, `BALANCED`, `OFF`).
* **Risk Level**: Payload safety level for active scanner (`SAFE`, `MODERATE`, `DANGEROUS`).
* **Agent Profile**: Markdown-based system instruction set selected from `~/.burp-ai-agent/AGENTS/`.
