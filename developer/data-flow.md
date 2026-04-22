# Data Flow

This page describes how data moves from Burp context to AI output, scanner findings, and MCP responses.

## Standard Chat Flow (Context Menu to AI Response)

```mermaid
flowchart TD
    Sel[User selection in Burp]\nRequest, response, or issue
    Act[Context action]\nFind vulnerabilities, Explain JS, etc.
    Col[ContextCollector]\ncollects selected data
    Red[Redaction Pipeline]\nSTRICT or BALANCED or OFF
    Cap[Manual context controls]\nbody caps and compact JSON
    Prompt[Prompt composition]\ntemplate and context
    Backend[Backend adapter]\nCLI or HTTP
    Chat[Chat panel]\nstreamed response
    Audit[Audit log]\nprompt and chunks

    Sel --> Col
    Act --> Col
    Col --> Red --> Cap --> Prompt --> Backend
    Backend --> Chat
    Backend --> Audit
```

**Operational notes**:

1. Context is collected from the selected Burp item(s).
2. Redaction is applied before any outbound AI call.
3. Manual context size controls are applied for context-menu actions.
4. Audit logging (when enabled) records prompt and stream events.

## BountyPrompt Flow (Request/Response Actions)

```mermaid
flowchart TD
    Start[User selects BountyPrompt action]
    Load[BountyPrompt loader]\ncurated JSON prompts and enabled IDs
    Resolve[Tag resolver]\nredaction-aware HTTP tags
    Compose[Prompt composer]\nsystem and user prompt
    Run[Backend stream to chat]
    Parse[Output parser]\nJSON extraction and fallback
    Gate{Confidence >= threshold?}
    Issue[Create Burp issue]\nAI/BountyPrompt prefix
    ChatOnly[Chat only]\nno issue creation

    Start --> Load --> Resolve --> Compose --> Run --> Parse --> Gate
    Gate -->|Yes| Issue
    Gate -->|No| ChatOnly
```

## Passive Scanner Flow

```mermaid
flowchart LR
    T[Proxy traffic]
    F[Filters]\nMIME, scope, size, stream patterns
    L[Local checks]\ncsrf, smuggling, upload, deserialization
    D[Dedup filters]\nendpoint and fingerprint windows
    C[Prompt cache lookup]
    Hit[Cached findings]
    AI[AI analysis]
    Conf{Confidence >= 85%?}
    Issue[Burp issue]\n[AI Passive]
    Queue{Auto-queue to active enabled?}
    ActiveQ[Active scanner queue]

    T --> F --> L --> D --> C
    C -->|Hit| Hit --> Conf
    C -->|Miss| AI --> Conf
    Conf -->|No| EndNoIssue[No issue creation]
    Conf -->|Yes| Issue --> Queue
    Queue -->|Yes| ActiveQ
    Queue -->|No| EndDone[Done]
```

### Batch Analysis & Persistent Cache Flow

```mermaid
flowchart TD
    Req[Request passes dedup]
    Batch{Batch size > 1?}
    Enqueue[Enqueue in BatchAnalysisQueue]
    Flush{Batch full or timeout?}
    BatchAI[Send batch prompt to AI]
    Dispatch[Dispatch findings by request_index]
    SingleAI[Send single prompt to AI]
    MemCache{In-memory cache hit?}
    DiskCache{Persistent cache hit?}
    Promote[Promote to in-memory]
    Store[Store in both caches]

    Req --> MemCache
    MemCache -->|Hit| Reuse[Reuse cached result]
    MemCache -->|Miss| DiskCache
    DiskCache -->|Hit| Promote --> Reuse
    DiskCache -->|Miss| Batch
    Batch -->|Yes| Enqueue --> Flush
    Flush -->|Yes| BatchAI --> Dispatch --> Store
    Flush -->|No| Wait[Wait for more requests]
    Batch -->|No| SingleAI --> Store
```

### Cross-Scanner Knowledge Base Flow

```mermaid
flowchart LR
    PH[Response headers] -->|Server, X-Powered-By| KB[ScanKnowledgeBase]
    PA[Passive findings] -->|VulnSignal| KB
    AF[Active confirmations] -->|VulnSignal + DB hints| KB
    KB -->|Tech stack, errors| AP[Adaptive Payload Engine]
    KB -->|Priority boost| AQ[Active scanner queue]
    KB -->|PRIOR KNOWLEDGE section| PP[Passive AI prompt]
```

## Active Scanner Flow

```mermaid
flowchart LR
    Target[Target request]
    Points[Injection point extraction]\nurl, body, header, cookie, json, xml, path
    Payloads[Payload selection]\nby risk level and scan mode
    Send[Request execution]\nrate and timeout controlled
    Analyze[Response analysis]\nerror, reflection, timing, OAST
    Confirm{Confirmed finding?}
    Issue[Burp issue]\n[AI Active]

    Target --> Points --> Payloads --> Send --> Analyze --> Confirm
    Confirm -->|Yes| Issue
    Confirm -->|No| NoIssue[No issue]
```

## MCP Tool Flow

```mermaid
flowchart LR
    Client[External MCP client]
    Transport[SSE or STDIO]
    Auth[Auth and host/origin validation]
    Limit[Request limiter]
    Handler[Tool handler]
    Burp[Burp API action]
    Privacy[Privacy filter]
    Resp[Tool response]
    Log[AI Request Logger]

    Client --> Transport --> Auth --> Limit --> Handler --> Burp --> Privacy --> Resp
    Handler --> Log
```

## Auto Tool Chaining Flow

When the AI needs to call MCP tools to answer a user question, tool calls are executed automatically in a loop:

```mermaid
flowchart TD
    User[User sends message]
    AI1[AI processes prompt]
    Check{Response contains tool call?}
    Parse[ToolCallParser extracts tool + args]
    Exec[Execute MCP tool]
    Log[Log to AI Request Logger with trace ID]
    Followup[Build follow-up prompt with tool result]
    Limit{Iteration <= 8?}
    AI2[AI processes follow-up]
    Final[Final response to user]

    User --> AI1 --> Check
    Check -->|No| Final
    Check -->|Yes| Parse --> Exec --> Log --> Followup --> Limit
    Limit -->|Yes| AI2 --> Check
    Limit -->|No| Final
```

All entries in a tool chain share the same trace ID (`chat-turn-{UUID}`), making it easy to follow the complete chain in the AI Request Logger.

## Trace ID Propagation

```mermaid
flowchart LR
    Chat[ChatPanel.sendMessage]
    Sup[AgentSupervisor.sendChat]
    Tool[maybeExecuteToolCall]
    Next[Recursive sendMessage]
    Logger[AI Request Logger]

    Chat -->|traceId| Sup
    Chat -->|traceId| Tool
    Tool -->|traceId| Logger
    Tool -->|traceId| Next
    Next -->|same traceId| Tool
```

Trace IDs are generated at the entry point and propagated through the entire call chain. Scanner jobs generate their own trace IDs (`scanner-job-{UUID}`).
