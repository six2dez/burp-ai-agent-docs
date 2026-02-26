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

## Troubleshooting

{% hint style="tip" %}
* Empty responses: run `ollama list` and verify model exists.
* Connection issues: confirm server process and URL.
* Slow output: reduce model size on limited hardware.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
