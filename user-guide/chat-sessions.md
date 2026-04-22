# Chat & Sessions

The chat interface is the primary way to interact with AI backends directly within Burp Suite.

## Sessions

Each chat session is an independent conversation with the AI. Sessions store:

* **Title**: Auto-generated from the first prompt or the context menu action used.
* **Creation time**: Timestamp for reference.
* **Last backend used**: The most recent backend used in that session.
* **Usage stats**: Message count, character totals, and per-backend distribution.
* **Token tracking**: Per-session input/output token counters with visual progress bars showing session vs. global token usage. Bars use color coding (green/yellow/orange) to indicate consumption level relative to global totals.

Session context is preserved per session. Follow-up prompts in the same session reuse the conversation history so the AI can keep track of previous responses and decisions.

History is trimmed to keep runtime bounded:

* HTTP backends: up to `20` messages and `40000` total characters (minimum latest 2 messages retained).
* CLI backends: up to `10` messages or `20000` total characters.

### Parallel Sessions

You can have multiple sessions open simultaneously, each using a different backend. For example:

* Session 1: Claude analyzing a complex authentication flow.
* Session 2: Ollama (local) analyzing API endpoints.

Switch between sessions using the session list in the left sidebar.

### Session Management

* **Rename**: Click the pencil icon to rename a session.
* **Delete**: Click the "X" icon or right‑click a session and select **Delete**.
* **Persistence**: Sessions are auto-saved and restored across Burp restarts.
* **Export**: Right‑click a session and choose **Export as Markdown**.

## Chat Interface

### Streaming Responses

AI responses stream in real time as tokens are generated. You see the response building incrementally rather than waiting for the full output.

### Markdown Rendering

AI responses are rendered as formatted Markdown, supporting:

* **Headers** (H1-H3)
* **Code blocks** and inline code
* **Blockquotes**
* **Lists** (ordered and unordered)
* **Bold/italic** text
* **Links**
* **Horizontal rules**

### Context Preview

Before the prompt is sent, you can see a preview of the redacted context. This shows exactly what data will leave Burp, allowing you to verify that sensitive information is properly handled.

### Error Display

If a backend fails or returns an error, the chat panel displays diagnostic information including:

* Error messages from the backend.
* Exit codes for CLI backends.
* Suggestions for common fixes.

### Cancel Requests

Use the **Cancel** button to stop an in‑flight response. A short system message is added to the chat to confirm cancellation.

## Tools Button

The **Tools** button opens a menu of available MCP tools. Selecting a tool inserts the corresponding command into the chat input field. Use this to:

* Force the AI to use a specific MCP tool.
* Build custom tool call sequences.
* Explore available tools without memorizing names.

## Auto Tool Chaining

When the AI determines it needs to call an MCP tool to answer a request, it executes the tool automatically and feeds the result back into the conversation. This process repeats until the AI has all the information it needs or the chain limit is reached.

* **Maximum chain depth**: 8 sequential tool calls per interaction.
* **Transparent execution**: Each tool call is logged in the [AI Request Logger](../privacy/ai-request-logger.md) with a shared trace ID so you can follow the full chain.
* **Automatic follow-up**: After each tool result, the AI continues reasoning with the original question plus all accumulated tool results. The final response is the complete answer, not intermediate tool output.
* **Works with all backends**: Tool chaining works with HTTP backends (Ollama, LM Studio, OpenAI-compatible) and CLI backends (Gemini, Claude, Codex, OpenCode).

### How It Works

1. You ask a question (e.g., "Generate a PoC for this issue").
2. The AI decides it needs data from Burp and calls an MCP tool (e.g., `proxy_http_history`).
3. The tool result is sent back to the AI along with the original question.
4. The AI may call another tool or produce the final answer.
5. Steps 2–4 repeat until the AI responds without a tool call, or 8 iterations are reached.

If the chain reaches the iteration limit, the AI produces a final response with whatever information it has gathered.

## Project-Scoped Storage

Chat sessions are stored in Burp's `extensionData()` API, which is scoped to the current Burp project file. This means each project maintains its own independent set of chat sessions, history, and token counters.

### Auto-Migration

On first open after this change, existing sessions stored in global `preferences()` are automatically migrated into the current project's `extensionData()`. No manual action is required.

### Project Change Detection

The extension polls `api.project().id()` every 30 seconds. When a project change is detected, the extension:

1. Saves the current project's chat sessions.
2. Clears in-memory chat state (messages, context, counters, drafts, tool state).
3. Restores sessions from the new project's stored data.
4. Clears the ScanKnowledgeBase to prevent cross-project data leakage.
5. Shuts down active chat backend connections.

### Clear Chat Behavior

The **Clear Chat** action fully resets the current session: messages, attached context, token counters, input drafts, tool call state, and all persisted data for that session.

## Tips for Effective Use

* **Be specific**: "Check this request for IDOR in the `user_id` parameter" is better than "find bugs."
* **Use context menu shortcuts**: The pre-built actions have optimized prompts that produce better results than freeform questions.
* **Iterate**: If the first response isn't detailed enough, ask follow-up questions in the same session — the AI keeps the conversation context per session.
* **Compare backends**: Try the same analysis with different backends to see which gives better results for your specific use case.
* **Use passive scanner for volume**: For large-scope assessments, let the passive scanner work in the background and use chat for deep-dive analysis on specific findings.

For token and cost tuning, see [Token Usage & Cost Management](token-management.md).
