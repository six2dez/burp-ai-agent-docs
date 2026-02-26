# FAQ

## Does it work with Burp Community Edition?

Yes. Core chat, context actions, passive scanner, and most MCP functionality work in Community. Pro-only capabilities (scanner APIs, some tools) remain gated.

## Does the passive scanner send extra traffic?

No. Passive scanner analyzes observed proxy traffic. Active scanner is the component that sends additional requests.

## Are my data safe with cloud backends?

Use `STRICT` or `BALANCED` privacy modes and review context before sending prompts. For maximum control, use local backends (Ollama/LM Studio).

## Which model/backend should I choose?

* Maximum privacy: local backends.
* Large cloud context: Gemini CLI.
* Reasoning-heavy workflows: Claude CLI.
* General purpose and code-heavy analysis: Codex CLI.

## How can I reduce cloud cost?

Tune passive scanner caps, dedup/cache settings, and context size controls. See [Token Usage & Cost Management](../user-guide/token-management.md).

## Can I use multiple backends at once?

Yes. Different chat sessions can use different backends. The backend selector controls new sessions.

## Why does backend status show Degraded/Offline?

Health checks detected command/API issues. Use backend **Test connection** and inspect extension output logs.

## Why are MCP tools missing?

Check tool toggles in **Burp Integration**, unsafe-tool master switch in **MCP Server**, and Burp edition constraints.
