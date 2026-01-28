# Audit Logging

Audit logs provide a **tamper-evident, reproducible record** of every interaction between Burp and the AI backend. This is essential for professional security audits, compliance, and debugging complex AI behavior.

## What is Logged?

Each event is recorded as a JSON object containing:
*   **Timestamp**: Precise time of the event.
*   **Event Type**: `prompt`, `agent_chunk`, `prompt_complete`, or scanner/audit events.
*   **Prompt Bundle**: The exact text sent to the AI (after redaction), plus hashes.
*   **Backend Metadata**: Backend ID, model settings, and determinism state.
*   **Response Chunks**: Streaming output captured as `agent_chunk` events.

## Log Format: JSONL
Logs are stored in **JSON Lines (.jsonl)** format. Each line is a standalone JSON object, making it easy to parse with tools like `jq` or import into a SIEM.

## Security & Integrity
To make logs tamper-evident:
*   Each event includes a SHA-256 hash of the payload.
*   When "Determinism Mode" is enabled, prompt bundles are stable for identical inputs, allowing hash comparison across runs.

## How to Enable

1.  Navigate to **Settings** → **Privacy & Logging**.
2.  Toggle **Audit Logging** to **ON**.

## File Locations

When audit logging is enabled, files are written to:

| Path | Contents |
| :--- | :--- |
| `~/.burp-ai-agent/audit.jsonl` | Main audit log. Each line is a JSON object with timestamp, event type, payload, and SHA-256 hash. |
| `~/.burp-ai-agent/bundles/` | Prompt bundle JSON files containing the full prompt text, context, backend config, and integrity hashes. |
| `~/.burp-ai-agent/contexts/` | Context snapshot JSON files (request/response data sent to the AI, indexed by SHA-256). |

The log path is not configurable. All files are stored under `~/.burp-ai-agent/`.

## Use Cases

*   **Compliance**: Demonstrating to a client that no PII was sent to a cloud AI.
*   **Reproducibility**: "Replaying" a complex AI-guided exploit to verify the steps.
*   **Quality Control**: Reviewing AI performance across a team of testers.
