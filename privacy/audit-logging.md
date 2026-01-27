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
3.  (Optional) Specify a custom log directory. By default, logs are saved to the extension's data folder.

![Screenshot: Audit logging toggle](../assets/screenshots/audit-logging.png)

## Use Cases

*   **Compliance**: Demonstrating to a client that no PII was sent to a cloud AI.
*   **Reproducibility**: "Replaying" a complex AI-guided exploit to verify the steps.
*   **Quality Control**: Reviewing AI performance across a team of testers.