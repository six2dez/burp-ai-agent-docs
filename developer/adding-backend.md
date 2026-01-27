# Adding a Backend

The extension uses Java's `ServiceLoader` mechanism for backend discovery, making it straightforward to add new AI backends without modifying core code.

## Steps

### 1. Implement the Backend Adapter

Create two classes:

*   **`AiBackendFactory`**: Factory that creates backend instances. Provides metadata (ID, display name, type).
*   **`AiBackend`**: The actual backend that implements `send()` to communicate with the AI.
*   **`AgentConnection`**: Interface for send/receive operations.

For HTTP backends, you can also implement `DiagnosableConnection` to expose exit codes and output for debugging.

### 2. Register via ServiceLoader

Create a file at:
```
META-INF/services/com.six2dez.burp.aiagent.backends.AiBackendFactory
```

Add the fully qualified class name of your factory:
```
com.example.mybackend.MyBackendFactory
```

### 3. Add UI Configuration Fields (Optional)

If your backend needs custom settings (API URL, model name, etc.), add configuration keys to `AgentSettings.kt` and corresponding UI fields to `SettingsPanel.kt`.

### 4. Ensure Audit Logging

Use the backend ID consistently so audit logs can trace which backend was used for each interaction.

### 5. Package and Deploy

**Option A: Built-in**
Add your backend to the main source tree and rebuild the extension JAR.

**Option B: Drop-in JAR**
Package your backend as a standalone JAR and place it in `~/.burp-ai-agent/backends/`. The `BackendRegistry` will discover it automatically via `URLClassLoader` on startup.

## Backend Interface

```kotlin
interface AiBackendFactory {
    val id: String           // Unique identifier (e.g., "my-backend")
    val displayName: String  // UI display name
    fun create(config: BackendLaunchConfig): AiBackend
}

interface AiBackend {
    fun send(prompt: String): AgentConnection
}

interface AgentConnection {
    fun receive(): String    // Blocking read of response
    fun close()              // Clean up resources
}
```

## Tips

*   Follow existing backend implementations as examples (e.g., `OllamaBackendFactory` for HTTP, `CodexCliBackendFactory` for CLI).
*   HTTP backends should handle connection errors and timeouts gracefully.
*   CLI backends should manage process lifecycle (stdin/stdout communication, exit handling).
