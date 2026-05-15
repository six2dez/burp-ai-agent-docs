# Best Practices & Hardening Checklist

A consolidated checklist for operators running Custom AI Agent against real targets. Each section is short and links to the canonical reference for parameter-level detail.

## Privacy Posture by Environment

Pick the privacy mode that matches the trust level of the **AI backend**, not the trust level of the target. Even an on-prem model crosses a trust boundary the moment its prompts leave Burp.

| Environment | Backend | Recommended mode | Why |
| :--- | :--- | :--- | :--- |
| Public bug bounty / customer engagement on a cloud LLM | Cloud HTTP (Perplexity, NVIDIA NIM, OpenAI-compatible) or Cloud CLI (Claude/Gemini/Codex/Copilot/OpenCode) | `STRICT` | Anonymizes hostnames and strips all tokens. Limits data leakage if the provider logs prompts. |
| Mixed workflow where the target hostname is part of the report context | Any cloud backend | `BALANCED` (default) | Preserves hostnames, redacts tokens and cookies. Cheap on prompt size, still safe. |
| Internal lab / private model with no third-party transit | Ollama, LM Studio, or Burp AI (Burp Pro built-in) | `BALANCED` for shared infra, `OFF` only for genuinely isolated single-user setups | Local does not mean unaccountable — keep redaction on unless you can prove no second copy of the prompts is being kept. |

Cross-references: [Privacy Modes](../privacy/privacy-modes.md), [Backends Overview](../backends/overview.md).

### Pre-engagement checks

- [ ] Privacy mode pill in the top bar matches the engagement's policy.
- [ ] Redaction salt is set (not the default placeholder) if anyone else on the team will need stable host pseudonyms.
- [ ] Determinism mode is enabled if you need reproducible prompt bundles for audit.
- [ ] Run one right-click action on a sample request and inspect the **Context Preview Dialog** before sending — confirm the JSON envelope matches expectations.

## MCP Hardening Checklist

The MCP server is **off by default**. Turn it on intentionally and review these before exposing it to anything beyond localhost.

- [ ] **Bind address** is `127.0.0.1` unless you genuinely need external access.
- [ ] If **External Access** is on:
  - [ ] **TLS** is enabled.
  - [ ] **Auto-Generate Certificate** is on **or** a custom keystore at `certs/mcp-keystore.p12` is in place with a non-default password.
  - [ ] Bearer **Token** has been rotated from any pre-shared value. Treat it like a credential — do not commit it.
  - [ ] **Allowed Origins** is populated with the explicit hostnames/origins of your clients. Empty means loopback-only.
- [ ] **Max Body Bytes** is sized to the largest legitimate tool response, not arbitrarily high.
- [ ] **Max Concurrent Requests** is sized to the host's capacity (default `4` is conservative).
- [ ] **Enable Unsafe Tools** is off unless an agent profile explicitly requires `http1_request`, `http2_request`, `intruder*`, `repeater_tab*`, scope mutation, or scanner-start tools.
- [ ] **Burp Integration** per-tool toggles match the agent profile in use — disable categories the profile never invokes.
- [ ] **Scan Task TTL** and **Collaborator Client TTL** are short enough that abandoned references do not accumulate (defaults `120` / `60` min).
- [ ] `/__mcp/health` returns `200` from the loopback before any external client is wired up.

Cross-references: [MCP Overview](../mcp/overview.md), [MCP Security Model](../mcp/security-model.md), [Tools Reference](../mcp/tools-reference.md).

## Active Scanner Safety

Active scanning sends real traffic to the target. The plugin enforces caps and gates, but the operator is the last line of defense.

- [ ] **Scope Only** is on (default). Confirm the in-scope filter in **Target → Scope** matches the engagement letter — not "everything I happened to proxy this morning".
- [ ] **Max Risk Level** is set to the lowest level that still produces signal for the engagement (`SAFE` for fragile prod, `MODERATE` for staging, `DANGEROUS` only for explicit, written authorization).
- [ ] **Max Concurrent Scans** is sized to the target's capacity. Default `3` is reasonable for typical web apps; raise only with the target owner's blessing.
- [ ] **Request Delay** is non-zero (default `100` ms) on shared/production infrastructure.
- [ ] **Use Collaborator (OAST)** is **on** only if Burp Collaborator is reachable from the engagement network. Off when running fully air-gapped.
- [ ] **AI Adaptive Payloads** is on only after baseline payloads have been validated — the adaptive layer can probe further than the operator expects.
- [ ] **Auto-Queue from Passive** is on if you trust the passive scanner's high-confidence findings to escalate; off if you want a manual gate between detection and exploitation.
- [ ] Before queueing many requests, glance at the queue size in the **AI Active Scanner** tab — the cap is `2000`, but anything near it suggests the previous run was not cleared.

Cross-references: [Active AI Scanner](../scanners/active.md), [Insertion Point Scan](../user-guide/insertion-point-scan.md), [Limitations & Hallucinations](../privacy/limitations.md).

## Audit Logging for Compliance

Audit logs are off by default and append-only when enabled. Use them when you need to prove what data left Burp.

- [ ] **Audit Logging** toggle is on for any engagement that has a written reporting requirement.
- [ ] `~/.burp-ai-agent/audit.jsonl` is on a volume with **enough headroom for the engagement** — JSONL grows with traffic.
- [ ] You have a backup script (or a cron-style copy) that pulls the JSONL file off the workstation before retention rotates it. The plugin does not rotate `audit.jsonl` itself.
- [ ] When running a sensitive engagement, also turn on **AI Request Logger** rolling persistence (`-Dburp.ai.logger.rolling.enabled=true`) so the chat/scanner activity stream is captured alongside the audit record.
- [ ] If you must show a third party that the prompts were redacted, freeze a prompt bundle (`bundles/`) for one representative action — bundles include the redacted payload and a SHA-256 hash.
- [ ] When the engagement closes, archive `audit.jsonl` + `bundles/` + relevant `contexts/` together. Each bundle is self-contained but references its context by hash.

Cross-references: [Audit Logging](../privacy/audit-logging.md), [AI Request Logger](../privacy/ai-request-logger.md).

## Cache Hygiene

The persistent prompt cache speeds re-scans dramatically and is project-scoped so it does not leak across engagements — but it is a copy of AI output that survives Burp restarts.

- [ ] Per-engagement, confirm `~/.burp-ai-agent/cache/<projectId>/` belongs to the active Burp project. Stray subdirectories from old projects are safe to delete.
- [ ] **Persistent max (MB)** is sized to the engagement — defaults to `50 MB`, which is fine for typical bug-bounty workflows but tight for long-running pentests.
- [ ] **Persistent TTL (hrs)** is short enough that you do not return stale findings on a re-scan after the target patched something (default `24 h` is a sensible ceiling).
- [ ] At engagement closeout, decide explicitly: keep the cache (faster re-scan if the customer asks for a re-test) or delete it (`rm -rf ~/.burp-ai-agent/cache/<projectId>/`) — there is no in-UI clear.
- [ ] If you swap models mid-engagement, remember the cache key is the *normalized prompt*, not the backend. Cached findings will be returned regardless of which backend produced them — clear the project's cache if you want to compare backends apples-to-apples.

Cross-references: [Passive AI Scanner → Cache Behavior](../scanners/passive.md#cache-key-prompt-hash), [Configuration Directory](configuration-directory.md).

## Backend Choice Trade-offs

Quick decision matrix when the engagement does not dictate a backend:

| Constraint | Pick |
| :--- | :--- |
| Lowest leakage, slowest model | Ollama or LM Studio with a local model. |
| Fastest cloud cycle, JSON-mode scanner workflows | OpenAI-compatible / NVIDIA NIM. |
| Web-aware reasoning, can give up JSON mode | Perplexity (Sonar family). |
| Already paying for Burp Pro AI credits | Burp AI (built-in) — no extra config. |
| Long context, code-heavy analysis | Gemini CLI or Claude CLI. |

Cross-reference: [Backends Overview](../backends/overview.md), [Backend Troubleshooting](../backends/troubleshooting.md).

## Related Pages

* [Privacy Modes](../privacy/privacy-modes.md)
* [MCP Security Model](../mcp/security-model.md)
* [Settings Reference](settings-reference.md)
* [Backend Troubleshooting](../backends/troubleshooting.md)
