# Ollama (Local)

Ollama is a popular tool for running LLMs locally. It is the recommended backend for users who cannot send data to the cloud due to privacy or compliance/NDA reasons.

## HTTP Mode (Recommended)

Connects directly to the Ollama HTTP API. This is faster and more reliable.

**Setup**:

1. Install Ollama from [ollama.com](https://ollama.com).
2.  Pull a model:

    ```bash
    ollama pull qwen2.5:14b-instruct
    ```
3.  Start the server (or let the desktop app run):

    ```bash
    ollama serve
    ```
4. Configure in Burp: open **Burp AI Agent → Settings → AI Backend** and set:

| Setting               | Value                                 |
| --------------------- | ------------------------------------- |
| **Preferred Backend** | `Ollama` (select from dropdown)       |
| **Ollama URL**        | `http://127.0.0.1:11434` (default)    |
| **Ollama Model**      | `qwen2.5:14b-instruct` (or any model) |

Optional: enable **Ollama Auto-Start** and set **Ollama Serve Command** to `ollama serve` so the extension starts the server automatically.

> **Note**: The **Ollama CLI Command** field is used to detect the default model (e.g., from `ollama run qwen2.5:14b-instruct`) and for launch validation. The active backend communicates with the Ollama HTTP API.

## Troubleshooting

* **Empty responses**: Ensure the model is actually downloaded (`ollama list`).
* **Slow performance**: Local inference depends heavily on your GPU/RAM. If you lack a GPU, stick to smaller models (7B or lower).
