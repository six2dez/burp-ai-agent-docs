# Adding a Backend

The extension uses Java's `ServiceLoader` mechanism for backend discovery, making it straightforward to add new AI backends without modifying core code.

## Steps

### 1. Implement the Backend Adapter

Implement a factory, a backend, and a connection:

* **`AiBackendFactory`**: zero-argument ServiceLoader factory whose `create()` returns the backend.
* **`AiBackend`**: owns `id`, `displayName`, availability/health checks, and `launch(config)`.
* **`AgentConnection`**: asynchronous send/stream/stop contract returned by `launch`.

CLI connections can also implement `DiagnosableConnection` to expose exit codes and output tails. HTTP connections commonly implement `UsageAwareConnection` and, when supported, `JsonModeCapable`.

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

If your backend needs custom settings (API URL, model name, etc.), add configuration keys to `AgentSettings.kt`, wire load/save in `SettingsPanelSettingsIO.kt`, and place the controls in the appropriate settings panel. If persisted shape changes, add a forward-only schema step as described in [Settings Migration](settings-migration.md).

### 4. Ensure Audit Logging

Use the backend ID consistently so audit logs can trace which backend was used for each interaction.

### 5. Package and Deploy

**Option A: Built-in**
Add your backend to the source tree and rebuild the extension JAR. A backend under `src/main` ships in both builds; a backend under `src/full`, listed in `src/full/resources/META-INF/services/com.six2dez.burp.aiagent.backends.AiBackendFactory`, ships in the GitHub full build only (this is where the CLI backends live).

**Option B: Drop-in JAR**

{% hint style="info" %}
**Full build only.** Drop-in backend JARs are part of the GitHub full build (`Custom-AI-Agent-full-<version>.jar`, built with `./gradlew fullJar`). The BApp Store build (`./gradlew bappJar`) does not include the drop-in loader: it loads no code from outside its JAR and never creates or reads `~/.burp-ai-agent/backends/`. See [Which Build?](../getting-started/which-build.md).
{% endhint %}

Package your backend as a standalone JAR and place it in `~/.burp-ai-agent/backends/`. In the full build, the drop-in loader (`DropInBackendJars`) creates that directory when it is missing and, on every backend registry reload, opens one `URLClassLoader` over the JARs in it and registers every `AiBackendFactory` its `ServiceLoader` finds.

## Backend Interface

```kotlin
class MyBackendFactory : AiBackendFactory {
    override fun create(): AiBackend = MyBackend()
}

class MyBackend : AiBackend {
    override val id = "my-backend"
    override val displayName = "My Backend"
    override val supportsSystemRole = true

    override fun launch(config: BackendLaunchConfig): AgentConnection =
        MyConnection(config)

    override fun isAvailable(settings: AgentSettings): Boolean = true

    override fun healthCheck(settings: AgentSettings): HealthCheckResult =
        HealthCheckResult.Healthy
}

class MyConnection(
    private val config: BackendLaunchConfig,
) : AgentConnection {
    override fun isAlive(): Boolean = true

    override fun send(
        text: String,
        history: List<ChatMessage>?,
        onChunk: (String) -> Unit,
        onComplete: (Throwable?) -> Unit,
        systemPrompt: String?,
        jsonMode: Boolean,
        maxOutputTokens: Int?,
    ) {
        // Run network/process work off the Swing EDT, call onChunk as data arrives,
        // then call onComplete exactly once.
    }

    override fun stop() {
        // Cancel I/O and release process/network resources.
    }
}
```

`BackendLaunchConfig` carries the resolved command or base URL, model, headers, timeouts, session IDs, determinism flag, context window, and shared Montoya HTTP transport. Implement `DiagnosableConnection`, `UsageAwareConnection`, `SessionAwareConnection`, `JsonModeCapable`, or `supportsSystemRole` only when the backend genuinely supports that capability.

## Tips

* Follow existing backend implementations as examples: `OllamaBackendFactory` for HTTP and `CodexCliBackendFactory` or `CopilotCliBackendFactory` for CLI (in `src/full`; CLI backends are full build only).
* Use the shared Montoya HTTP transport and retry/circuit-breaker support instead of creating an unrelated client stack.
* Never block the Swing EDT. The connection callback contract is asynchronous.
* Ensure `onComplete` fires exactly once on success, error, cancellation, and timeout.
* CLI backends must manage stdin/stdout, exit status, temporary files, cancellation, and Windows `.cmd` shims.
