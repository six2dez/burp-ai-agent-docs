# Audit Logging

Audit logs provide a **tamper-evident, reproducible record** of every interaction between Burp and the AI backend. This is essential for professional security audits, compliance, and debugging complex AI behavior.

## What is Logged?

Each interaction is recorded as a JSON object containing:
*   **Timestamp**: Precise time of the request/response.
*   **Prompt Bundle**: The exact text sent to the AI (after redaction).
*   **Context Hashes**: SHA-256 hashes of the attached requests/responses to ensure the data hasn't been modified.
*   **Backend Metadata**: Backend ID (e.g., `gemini-cli`), model name, and temperature settings.
*   **Transcript**: The AI's response and any tool calls made.

## Log Format: JSONL
Logs are stored in **JSON Lines (.jsonl)** format. Each line is a standalone JSON object, making it easy to parse with tools like `jq` or import into a SIEM.

## Security & Integrity
To ensure logs haven't been tampered with:
*   The extension can generate a **Digest** of the log file.
*   When "Determinism Mode" is enabled, you can re-run a session and verify that the prompt hashes match the audit log exactly.

## How to Enable

1.  Navigate to **Settings** → **Privacy & Logging**.
2.  Toggle **Audit Logging** to **ON**.

![Screenshot: Audit logging toggle](../assets/screenshots/audit-logging.png)

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