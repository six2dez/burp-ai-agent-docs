# Feature Coverage Matrix

This matrix tracks documentation coverage for features currently staged in the plugin repository.

| Feature | Primary Doc Page | Coverage Status |
| :--- | :--- | :--- |
| Passive scanner dedup/cache/token controls | [Passive AI Scanner](../scanners/passive.md), [Settings Reference](../reference/settings-reference.md) | Documented |
| Manual context size controls + compact JSON | [Context Menus](../user-guide/context-menus.md), [Settings Reference](../reference/settings-reference.md) | Documented |
| Prompt-result cache behavior | [Passive AI Scanner](../scanners/passive.md) | Documented |
| Token usage telemetry | [Token Usage & Cost Management](../user-guide/token-management.md), [Chat & Sessions](../user-guide/chat-sessions.md) | Documented |
| Backend health states and diagnostics UX | [UI Tour](../user-guide/ui-tour.md), [Troubleshooting](../reference/troubleshooting.md) | Documented |
| Active scanner queue panel and backpressure | [Active AI Scanner](../scanners/active.md) | Documented |
| Conversation history dual-budget trimming | [Chat & Sessions](../user-guide/chat-sessions.md), [Settings Reference](../reference/settings-reference.md) | Documented |
| BountyPrompt category-specific context limits | [BountyPrompt Actions](../user-guide/bountyprompt-actions.md) | Documented |
| Auto tool chaining (8 iterations) | [Chat & Sessions](../user-guide/chat-sessions.md), [Data Flow](data-flow.md) | Documented |
| AI Request Logger (UI + buffer) | [AI Request Logger](../privacy/ai-request-logger.md), [UI Tour](../user-guide/ui-tour.md) | Documented |
| Rolling JSONL persistence | [AI Request Logger](../privacy/ai-request-logger.md), [Settings Reference](../reference/settings-reference.md) | Documented |
| Trace ID correlation | [AI Request Logger](../privacy/ai-request-logger.md), [Audit Logging](../privacy/audit-logging.md), [Data Flow](data-flow.md) | Documented |
| Backend retry with diagnostics logging | [Backends Overview](../backends/overview.md), [Ollama](../backends/ollama.md) | Documented |
| MCP tool metadata (policy, hashes, status) | [AI Request Logger](../privacy/ai-request-logger.md), [MCP Tools Reference](../mcp/tools-reference.md) | Documented |
| Agent profile optional tool handling | [Agent Profiles](../user-guide/agent-profiles.md) | Documented |
| ToolCallParser (multi-format extraction) | [Architecture](architecture.md) | Documented |
| Copilot CLI backend | [Copilot CLI](../backends/copilot-cli.md), [Backends Overview](../backends/overview.md) | Documented |
| System prompt support for HTTP backends | [Chat & Sessions](../user-guide/chat-sessions.md), [Architecture](architecture.md) | Documented |
| Per-session token tracking + visual bars | [Chat & Sessions](../user-guide/chat-sessions.md) | Documented |
| Context collection global size cap | [Settings Reference](../reference/settings-reference.md) | Documented |
| Passive scanner prompt improvements | [Passive AI Scanner](../scanners/passive.md) | Documented |
| System proxy support for HTTP backends | [Backends Overview](../backends/overview.md) | Documented |
| Chat context dedup (contextSent) | [Chat & Sessions](../user-guide/chat-sessions.md) | Implicit |
| Issue consolidation canonical names | [Architecture](architecture.md) | Implicit |
| Pre-release stability hardening (3 rounds) | [Changelog](../reference/changelog.md) | Documented |
| Settings schema migration | [Settings Reference](../reference/settings-reference.md), [Changelog](../reference/changelog.md) | Partial |
| MCP hardening/operator runbooks | [MCP Security Model](../mcp/security-model.md) | Partial |

## Coverage Gaps to Track

* Add explicit schema migration section with example migration behavior.
* Add dedicated operator runbook pages if plugin repo runbooks are reinstated.
