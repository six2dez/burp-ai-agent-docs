# Data Flow

This page describes how data moves through the extension, from user action to AI response.

## Chat Flow (Context Menu → AI Response)

```
User right-clicks request        Context menu action triggered
        │                                    │
        ▼                                    ▼
┌───────────────┐              ┌──────────────────────┐
│ Burp Selection │──────────►  │  ContextCollector     │
│ (request/issue)│              │  Extracts URL, method,│
└───────────────┘              │  headers, params, body│
                                └──────────┬───────────┘
                                           │
                                           ▼
                                ┌──────────────────────┐
                                │  Redaction Pipeline   │
                                │  Applies privacy mode │
                                │  (STRICT/BALANCED/OFF)│
                                └──────────┬───────────┘
                                           │
                                           ▼
                                ┌──────────────────────┐
                                │  Prompt Bundle        │
                                │  Context + template + │
                                │  SHA-256 hash         │
                                └──────────┬───────────┘
                                           │
                                           ▼
                                ┌──────────────────────┐
                                │  Backend Adapter      │
                                │  CLI (subprocess) or  │
                                │  HTTP (API call)      │
                                └──────────┬───────────┘
                                           │
                                    ┌──────┴──────┐
                                    ▼             ▼
                            ┌────────────┐ ┌────────────┐
                            │ Audit Log  │ │ Chat Panel │
                            │ (JSONL)    │ │ (Markdown) │
                            └────────────┘ └────────────┘
```

## Step-by-Step

1.  **User selects context**: Right-click on a request/response in Proxy, Repeater, or Site Map; or on an issue in the Scanner panel.
2.  **Context collection**: `ContextCollector` extracts structured data from the Burp selection — URL, method, headers, parameters, body, and (for issues) severity, confidence, and remediation.
3.  **Redaction**: The `Redaction` module applies the active privacy mode:
    *   **STRICT**: Strips cookies, redacts auth tokens, anonymizes hostnames using SHA-256(salt + host).
    *   **BALANCED**: Strips cookies, redacts auth tokens, preserves hostnames.
    *   **OFF**: No modification.
4.  **Prompt bundle**: The redacted context is combined with the selected prompt template. If determinism is enabled, context ordering is stabilized. A SHA-256 hash of the bundle is computed for audit integrity.
5.  **Backend processing**: The prompt is sent to the configured AI backend (CLI subprocess or HTTP API). The response streams back in real time.
6.  **Audit logging**: If enabled, the prompt bundle, context hashes, backend metadata, and AI transcript are written to `~/.burp-ai-agent/audit.jsonl`.
7.  **Display**: The AI response is rendered as Markdown in the chat panel. For scanner findings, issues may be created automatically.

## Passive Scanner Flow

```
Proxy traffic ──► MIME filter ──► Scope filter ──► Size filter ──► Rate limiter ──► AI Analysis ──► Findings
                                                                                                     │
                                                                                        (confidence ≥ 85%)
                                                                                                     │
                                                                                                     ▼
                                                                                              Burp Issues [AI]
                                                                                                     │
                                                                                          (if auto-queue enabled)
                                                                                                     │
                                                                                                     ▼
                                                                                            Active Scanner Queue
```

## Active Scanner Flow

```
Target request ──► Injection Point Extraction ──► Payload Selection ──► Send Payloads ──► Response Analysis
                        │                              │                                        │
                   URL params               Risk level + scan mode                     Detection method
                   Body params              Vuln class filter                          (error/blind/time/
                   Headers                                                              reflection/OOB)
                   Cookies                                                                      │
                   JSON fields                                                                  ▼
                   XML elements                                                        Confirmed findings
                   Path segments                                                       with [AI] prefix
```

## MCP Tool Flow

```
External AI Agent ──► SSE/STDIO ──► Token Auth ──► Tool Gating ──► Request Limiter ──► Tool Handler
        │                                                                                    │
   (Claude Desktop,                                                                   Burp API call
    custom scripts)                                                                  (history, repeater,
                                                                                      scanner, etc.)
                                                                                          │
                                                                                          ▼
                                                                                   Privacy filter
                                                                                          │
                                                                                          ▼
                                                                                   Response to agent
```
