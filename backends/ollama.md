# Ollama (Local)

Ollama is a popular tool for running LLMs locally. It is the recommended backend for users who cannot send data to the cloud due to privacy or compliance/NDA reasons.

## Modes

The extension supports Ollama in two ways:

### 1. HTTP Mode (Recommended)
Connects directly to the Ollama HTTP API. This is faster and more reliable.

**Setup**:
1.  Install Ollama from [ollama.com](https://ollama.com).
2.  Pull a model:
    ```bash
    ollama pull llama3.1
    ```
3.  Start the server (or let the desktop app run):
    ```bash
    ollama serve
    ```
4.  Configure in Burp: open **Burp AI Agent → Settings → AI Backend** and set:

| Setting | Value |
| :--- | :--- |
| **Preferred Backend** | `Ollama` (select from dropdown) |
| **Ollama URL** | `http://127.0.0.1:11434` (default) |
| **Ollama Model** | `llama3.1` (or any model you pulled) |

Optional: enable **Ollama Auto-Start** and set **Ollama Serve Command** to `ollama serve` so the extension starts the server automatically.

### 2. CLI Mode
Spawns `ollama run` as a subprocess in a terminal window.

**Setup**: In **Settings → AI Backend**, set:

| Setting | Value |
| :--- | :--- |
| **Ollama CLI Command** | `ollama run llama3.1` (includes the model name) |

The extension interacts with this process via stdin/stdout. The model name is extracted from the command automatically.

## Troubleshooting

*   **Empty responses**: Ensure the model is actually downloaded (`ollama list`).
*   **Slow performance**: Local inference depends heavily on your GPU/RAM. If you lack a GPU, stick to smaller models (7B or lower).