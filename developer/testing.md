# Testing & Debugging

## Running Tests

The project uses JUnit 5 for testing.

### Unit Tests
Run the standard test suite:
```bash
./gradlew test
```

### Integration Tests
Some tests require a local Ollama instance or mock server. Ensure your environment is set up if running the full suite.

## Building from Source

To build the extension JAR:
```bash
./gradlew clean shadowJar
```
The output JAR will be in `build/libs/Burp-AI-Agent-<version>.jar`. Use that JAR to install the extension in Burp.

## Debugging

If the extension is behaving unexpectedly, check the logs.

### 1. Burp Extension Output
In Burp Suite:
1.  Go to **Extensions** → **Installed**.
2.  Select **Burp AI Agent**.
3.  Click the **Output** tab.
    *   **Standard Output**: Shows general info and initialization messages.
    *   **Errors**: Shows Java stack traces if the extension crashes.

### 2. Java Logs
The extension uses SLF4J. Logs are typically printed to the Burp stdout.
If you launched Burp from a terminal, check the terminal window for detailed logs.

### 3. Agent Debugging
If a backend is failing:
1.  Check the **Status** indicator in the top bar.
2.  If it says **Crashed**, check **Extensions → Installed → Burp AI Agent → Output/Errors** for the exit code.
3.  Check the `audit.jsonl` file (if Audit Logging is enabled) to see exactly what was sent to the model.

## Manual UI Checks
Before a release, perform these sanity checks:
- [ ] Load extension in Burp Community and Pro.
- [ ] Verify tab appearance on macOS (Retina), Linux, and Windows (HiDPI).
- [ ] Check context menus appear on Request/Response and Issues.
- [ ] Verify scrolling in the Chat interface.
