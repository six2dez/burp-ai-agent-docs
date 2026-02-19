# Data Flow

This page describes data movement from Burp selection to AI output.

## Standard Chat Flow (Context Menu -> AI Response)

```
User selection in Burp              Context action triggered
        │                                      │
        ▼                                      ▼
┌────────────────┐                 ┌──────────────────────┐
│ Request / Issue │──────────────► │ ContextCollector      │
└────────────────┘                 └──────────┬───────────┘
                                              │
                                              ▼
                                   ┌──────────────────────┐
                                   │ Redaction Pipeline    │
                                   │ STRICT/BALANCED/OFF  │
                                   └──────────┬───────────┘
                                              │
                                              ▼
                                   ┌──────────────────────┐
                                   │ Prompt Composition    │
                                   │ Template + context    │
                                   └──────────┬───────────┘
                                              │
                                              ▼
                                   ┌──────────────────────┐
                                   │ Backend Adapter       │
                                   │ CLI or HTTP           │
                                   └──────┬─────────┬─────┘
                                          │         │
                                          ▼         ▼
                                  ┌────────────┐ ┌────────────┐
                                  │ Audit Log  │ │ Chat Panel │
                                  └────────────┘ └────────────┘
```

## BountyPrompt Flow (Request/Response Actions)

```
User selects BountyPrompt action
            │
            ▼
┌───────────────────────────────┐
│ BountyPrompt Loader           │
│ - reads curated JSON prompts  │
│ - filters by enabled IDs      │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ Tag Resolver                  │
│ - applies privacy redaction   │
│ - resolves [HTTP_*] tokens    │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ Prompt Composer               │
│ - systemPrompt + user task    │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ Backend + Chat Stream         │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ Output Parser (if issue type) │
│ - JSON extraction / fallback   │
│ - NONE guard                   │
│ - confidence threshold         │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ Burp Issue Creation           │
│ [AI][BountyPrompt] prefix     │
└───────────────────────────────┘
```

## Step-by-Step Sequence

1. User selects context (request/response or issue).
2. `ContextCollector` extracts raw context plus metadata.
3. Redaction applies active privacy mode.
4. Action prompt is composed.
5. Backend processes prompt and streams output.
6. Audit logging records invocation/output events if enabled.
7. Response is rendered in chat.
8. For eligible BountyPrompt outputs, parser and confidence gate decide issue creation.

## Passive Scanner Flow

```
Proxy traffic -> MIME filter -> Scope filter -> Size filter -> Rate limiter -> AI analysis -> Findings
                                                                                           │
                                                                                   confidence >= 85%
                                                                                           │
                                                                                           ▼
                                                                                   Burp Issues [AI]
                                                                                           │
                                                                                 (optional auto-queue)
                                                                                           │
                                                                                           ▼
                                                                                     Active queue
```

## Active Scanner Flow

```
Target request -> Injection point extraction -> Payload selection -> Send payloads -> Response analysis
                       │                             │                                      │
                       ▼                             ▼                                      ▼
             params/headers/body/path       risk level + mode                         confirmed findings
```

## MCP Tool Flow

```
External MCP client -> SSE/STDIO -> auth/gating -> limiter -> tool handler -> Burp API -> response
```
