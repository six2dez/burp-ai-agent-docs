# LM Studio (Local)

LM Studio provides local model execution with an OpenAI-compatible API and a GUI model manager.

## Requirements

* LM Studio installed.
* A model downloaded and loaded.

## Setup

1. Install LM Studio from [lmstudio.ai](https://lmstudio.ai).
2. Load a model.
3. Start local server (`Developer -> Start Server`).
4. Configure in **AI Backend** settings tab.

## Configuration

| Setting | Default | Description |
| :--- | :--- | :--- |
| **Preferred Backend** | `LM Studio` | Select backend. |
| **LM Studio URL** | `http://127.0.0.1:1234` | Server API base URL. |
| **LM Studio Model** | `lmstudio` | Model identifier. |
| **LM Studio API Key** | *(empty)* | Optional bearer token. |
| **LM Studio Headers** | *(empty)* | Extra headers (`Header: value`). |
| **LM Studio Auto-Start** | On | Start server command automatically. |
| **LM Studio Server Command** | `lms server start` | Auto-start command. |
| **LM Studio Timeout** | `120` | Request timeout in seconds. |

## Notes

Use Auto-Start only if your LM Studio server command is stable in the same runtime environment as Burp.

## Troubleshooting

{% hint style="tip" %}
* Connection refused: verify server is running and URL/port match.
* Model not found: confirm model ID from LM Studio server logs.
* Timeouts: increase timeout or use smaller model.
* Slow responses: local hardware constraints are expected on CPU-only setups.
{% endhint %}

## Related Pages

* [Backends Overview](overview.md)
* [Troubleshooting](../reference/troubleshooting.md)
