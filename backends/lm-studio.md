# LM Studio (Local)

LM Studio is a desktop application for running local LLMs with a user-friendly interface. It provides an OpenAI-compatible API server, making it a good option for users who prefer a GUI for model management.

## Modes

The extension supports LM Studio in two ways:

### 1. HTTP Mode (Recommended)

Connects directly to the LM Studio local server API.

**Setup**:

1. Download and install LM Studio from [lmstudio.ai](https://lmstudio.ai).
2. Download a model within LM Studio.
3. Start the local server: **Developer** tab → **Start Server**.
4. Configure in Burp: open **Burp AI Agent → AI Backend tab in the bottom settings panel** and set:

| Setting               | Value                                                                 |
| --------------------- | --------------------------------------------------------------------- |
| **Preferred Backend** | `LM Studio` (select from dropdown)                                    |
| **LM Studio URL**     | `http://127.0.0.1:1234` (default)                                     |
| **LM Studio Model**   | The identifier of the model you loaded (check LM Studio's server log) |
| **LM Studio API Key** | *(optional)* `Bearer` token                                           |
| **LM Studio Headers** | *(optional)* Extra headers                                            |

### 2. Auto-Start Mode

The extension can launch the LM Studio server for you.

**Setup**: In **AI Backend tab in the bottom settings panel**, set:

| Setting                      | Value                                        |
| ---------------------------- | -------------------------------------------- |
| **LM Studio Auto-Start**     | ON                                           |
| **LM Studio Server Command** | The command that starts the LM Studio server |

When Auto-Start is enabled, the extension runs the server command automatically when this backend is selected.

## Configuration

| Setting             | Default                 | Description                                                                               |
| ------------------- | ----------------------- | ----------------------------------------------------------------------------------------- |
| **LM Studio URL**   | `http://127.0.0.1:1234` | Base URL for the LM Studio server API.                                                    |
| **LM Studio Model** | `lmstudio`              | Model identifier to use.                                                                  |
| **Auto-Start**      | On                      | Automatically start the LM Studio server.                                                 |
| **Server Command**  | `lms server start`      | Custom command to launch the server.                                                      |
| **Timeout**         | 120 seconds             | Request timeout (range: 30–3600 seconds). Increase for larger models or complex analyses. |
| **API Key**         | *(empty)*               | Optional `Authorization: Bearer` token for OpenAI-compatible servers.                     |
| **Extra Headers**   | *(empty)*               | Extra headers, one per line (`Header: value`).                                            |

## Troubleshooting

* **Connection refused**: Ensure the LM Studio server is running and the port matches your configuration.
* **Model not found**: Verify the model identifier matches the loaded model in LM Studio.
* **Timeout errors**: Increase the **Timeout** setting for large responses. Consider using a smaller model if timeouts persist.
* **Slow responses**: LM Studio performance depends on your hardware. GPU inference is significantly faster than CPU-only.
