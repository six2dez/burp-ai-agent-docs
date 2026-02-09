# Chat & Sessions

The chat interface is the primary way to interact with AI backends directly within Burp Suite.

## Sessions

Each chat session is an independent conversation with the AI. Sessions store:

* **Title**: Auto-generated from the first prompt or the context menu action used.
* **Creation time**: Timestamp for reference.
* **Last backend used**: The most recent backend used in that session.
* **Usage stats**: Message count and character totals, visible in the sidebar summary.

Session context is preserved per session. Follow-up prompts in the same session reuse the conversation history so the AI can keep track of previous responses and decisions.

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

## Tips for Effective Use

* **Be specific**: "Check this request for IDOR in the `user_id` parameter" is better than "find bugs."
* **Use context menu shortcuts**: The pre-built actions have optimized prompts that produce better results than freeform questions.
* **Iterate**: If the first response isn't detailed enough, ask follow-up questions in the same session — the AI keeps the conversation context per session.
* **Compare backends**: Try the same analysis with different backends to see which gives better results for your specific use case.
* **Use passive scanner for volume**: For large-scope assessments, let the passive scanner work in the background and use chat for deep-dive analysis on specific findings.
