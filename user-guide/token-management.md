# Token Usage & Cost Management

This page explains how Burp AI Agent tracks usage and how to reduce cost/noise when using cloud backends.

## What Is Tracked

Session statistics include:

* input/output character totals,
* estimated token usage,
* per-backend usage distribution.

Passive scanner flow also records token-aware telemetry to help tune background analysis settings.

## Where to View Usage

* Session sidebar summary in the chat panel.
* Passive scanner metrics in the scanner settings area.
* Audit logs (if enabled) for detailed event-level analysis.

## Conversation History Budgets

To prevent prompt growth over long sessions:

| Backend Type | Budget |
| :--- | :--- |
| HTTP | `20` messages + `40000` total chars (minimum latest 2 kept) |
| CLI | `10` messages or `20000` total chars |

## Cost Reduction Controls

Use these settings in **AI Passive Scanner** tab:

* `Rate Limit`
* `Max Size (KB)`
* `Req body chars (AI)` / `Resp body chars (AI)`
* `Max headers` / `Max params`
* `Endpoint dedup (min)` / `Response dedup (min)`
* `Prompt cache TTL (min)`
* `Manual context JSON` (compact)

## Practical Tuning Strategy

1. Keep **Scope Only** ON.
2. Start with defaults.
3. If cost is high, reduce response body cap first.
4. Then reduce headers/params and max size.
5. Increase dedup/cache windows for repetitive traffic.
6. For high-volume targets, increase `Rate Limit`.

## Provider Cost Estimation Tips

* Character counts are estimates, not provider-billed exact tokens.
* Different providers tokenize text differently.
* Use your provider billing dashboard for final spend validation.

## Related Pages

* [Chat & Sessions](chat-sessions.md)
* [Passive AI Scanner](../scanners/passive.md)
* [Settings Reference](../reference/settings-reference.md)
