# Audit Logging

Audit logs provide a tamper-evident record of interactions between Burp context and AI backend outputs.

## Event Chain

```mermaid
sequenceDiagram
    participant User as User Action
    participant App as Burp AI Agent
    participant Hash as SHA-256 Hasher
    participant Log as audit.jsonl

    User->>App: Trigger action (chat/scanner/MCP)
    App->>App: Build redacted prompt bundle
    App->>Hash: Hash payload/event data
    Hash-->>App: Digest
    App->>Log: Append JSONL event line
    App->>Log: Append streaming chunks/events
    App->>Log: Append completion event
```

## What Is Logged

Each event entry can include:

* timestamp,
* event type (`prompt`, `agent_chunk`, `prompt_complete`, scanner/MCP events),
* redacted prompt bundle and hashes,
* backend metadata,
* streamed response chunks.

## Log Format

Logs use **JSON Lines (`.jsonl`)**; each line is a standalone JSON object.

## Security & Integrity

* Each event includes SHA-256 payload hashing.
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

## Use Cases

* Compliance evidence.
* Reproducibility and review of AI-assisted findings.
* Team quality control and diagnostics.

## Related Pages

* [Privacy Modes](privacy-modes.md)
* [Determinism & Salt](determinism-salt.md)
