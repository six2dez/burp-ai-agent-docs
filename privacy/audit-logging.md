# Audit Logging

Audit logs provide a tamper-evident record of interactions between Burp context and AI backend outputs.

## Event Chain

```mermaid
sequenceDiagram
    participant User as User Action
    participant App as Custom AI Agent
    participant Hash as SHA-256 Hasher
    participant Log as audit.jsonl
    participant Logger as AI Request Logger

    User->>App: Trigger action (chat/scanner/MCP)
    App->>App: Build redacted prompt bundle
    App->>Hash: Hash payload/event data
    Hash-->>App: Digest
    App->>Log: Append JSONL event line
    App->>Logger: Log activity entry with trace ID
    App->>Log: Append streaming chunks/events
    App->>Log: Append completion event
    App->>Logger: Log completion with duration and metadata
```

## What Is Logged

Each event entry can include:

* timestamp,
* event type (`prompt`, `agent_chunk`, `prompt_complete`, scanner/MCP events),
* trace ID for correlation across related entries,
* redacted prompt bundle and hashes,
* backend metadata,
* streamed response chunks,
* launch metadata (see below).

## Launch Metadata

Every context-driven chat launch (right-click → any AI action) stamps the resulting `prompt` event and the `PromptBundle` saved under `bundles/` with:

| Field | Values | Meaning |
| :--- | :--- | :--- |
| `promptSource` | `FIXED` \| `CUSTOM_SAVED` \| `CUSTOM_AD_HOC` | Origin of the prompt. Canned actions report `FIXED`; saved library entries report `CUSTOM_SAVED`; free-form one-offs report `CUSTOM_AD_HOC`. |
| `contextKind` | `HTTP_SELECTION` \| `SCANNER_ISSUE` | Which menu the launch came from. |
| `promptId` | UUID | Identifier of the saved custom prompt (only for `CUSTOM_SAVED`). |
| `promptTitle` | string | Human-readable title of the saved custom prompt (only for `CUSTOM_SAVED`). |

These also land in the `metadata` map of matching `AiRequestLogger` entries (PROMPT_SENT, RESPONSE_COMPLETE, ERROR).

Example filter for recent custom-prompt runs:

```
tail -n 500 ~/.burp-ai-agent/audit.jsonl \
  | jq 'select(.type=="prompt" and .payload.promptSource!="FIXED")
        | {ts, title:.payload.promptTitle, source:.payload.promptSource, ctx:.payload.contextKind}'
```

## Trace ID Correlation

Every operation generates a trace ID that links all related log entries:

* **Chat interactions**: `chat-turn-{UUID}` — links the prompt, any tool chain steps, and the final response.
* **Agent supervisor calls**: `agent-turn-{UUID}` — links prompt dispatch and result.
* **Scanner jobs**: `scanner-job-{UUID}` — links analysis dispatch and outcome.

Trace IDs are visible in both the audit log entries and the [AI Request Logger](ai-request-logger.md) UI.

## Log Format

Logs use **JSON Lines (`.jsonl`)**; each line is a standalone JSON object.

## Security & Integrity

* Each event includes SHA-256 payload hashing.
* MCP tool calls include argument and result hashes (`argsSha256`, `resultSha256`).
* With determinism enabled, identical inputs are easier to compare across runs.

## How to Enable

1. Open **Privacy & Logging** tab in Settings.
2. Toggle **Audit Logging** ON.

## File Locations

| Path | Contents |
| :--- | :--- |
| `~/.burp-ai-agent/audit.jsonl` | Main append-only event log. |
| `~/.burp-ai-agent/bundles/` | Prompt bundle snapshots. |
| `~/.burp-ai-agent/contexts/` | Context snapshot files indexed by hash. |
| `~/.burp-ai-agent/logs/` | Rolling AI Request Logger JSONL files (opt-in). |

## Use Cases

* Compliance evidence.
* Reproducibility and review of AI-assisted findings.
* Team quality control and diagnostics.
* Trace ID-based debugging of multi-step tool chains.

## Related Pages

* [AI Request Logger](ai-request-logger.md)
* [Privacy Modes](privacy-modes.md)
* [Determinism & Salt](determinism-salt.md)
