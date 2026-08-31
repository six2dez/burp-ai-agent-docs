# Ollama (Local)

Ollama is the recommended local backend when privacy constraints prevent cloud usage.

## Requirements

* Ollama installed.
* At least one model pulled locally.

## Setup

1. Install Ollama from [ollama.com](https://ollama.com).
2. Pull a model:

```bash
ollama pull qwen2.5:14b-instruct
```

3. Start server:

```bash
ollama serve
```

4. Configure in **AI Backend** settings tab.

## Configuration

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `ollama` |
| **Ollama CLI Command** | `ollama run llama3.1` (default; used for validation/model fallback) |
| **Ollama URL** | `http://127.0.0.1:11434` |
| **Ollama Model** | `llama3.1` by default; e.g. `qwen2.5:14b-instruct` |
| **Ollama API Key** | optional |
| **Ollama Headers** | optional |
| **Ollama Auto-Start** | optional (`ollama serve`) |

## Notes

The active backend path is HTTP-based. The CLI command is used for validation and as a fallback model source when the explicit Ollama model is blank.

## Output Token Limits

The extension now sets `num_predict` automatically per request type. Previously, Ollama defaulted to 128 tokens, which caused truncated responses. Current values:

| Request Type | `num_predict` |
| :--- | :--- |
| **Chat** | 4096 |
| **Scanner (single request)** | 2048 |
| **Scanner (batch analysis)** | 4096 |
| **Payload generation** | 1024 |

## Troubleshooting

{% hint style="tip" %}
* Empty responses: run `ollama list` and verify model exists.
* Connection issues: confirm server process and URL.
* Slow output: reduce model size on limited hardware.
{% endhint %}

## Retry Behavior

If a request to Ollama throws a classified transient transport exception (for example, connection timeout/refused), the extension makes up to 6 total attempts. The five possible retry delays are 500/1000/1500/2000/3000 ms. Each retry is logged in the [AI Request Logger](../privacy/ai-request-logger.md) with the attempt number and delay. The connection's circuit breaker opens after 5 recorded transient failures and allows one half-open model request after 30 seconds — see [Backends Overview → Retry Behavior](overview.md#retry-behavior).

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
