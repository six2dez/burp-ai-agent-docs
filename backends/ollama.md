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
| **Preferred Backend** | `Ollama` |
| **Ollama URL** | `http://127.0.0.1:11434` |
| **Ollama Model** | e.g. `qwen2.5:14b-instruct` |
| **Ollama API Key** | optional |
| **Ollama Headers** | optional |
| **Ollama Auto-Start** | optional (`ollama serve`) |

## Notes

The active backend path is HTTP-based. CLI command fields are primarily used for validation/default-model detection.

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

If a request to Ollama fails due to a transient network error (connection timeout, connection refused), the extension retries automatically with exponential backoff up to 6 attempts. Each retry is logged in the [AI Request Logger](../privacy/ai-request-logger.md) with the attempt number and delay.

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
