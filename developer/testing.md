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
The output JAR will be in `build/libs/burp-ai-agent-all.jar`.

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
The extension uses SLF4J/Logback. Logs are typically printed to the Burp stdout.
If you launched Burp from a terminal, check the terminal window for detailed logs.

### 3. Agent Debugging
If a backend is failing:
1.  Check the **Status** indicator in the top bar.
2.  If it says **Crashed**, click it to see the exit code.
3.  Check the `audit.jsonl` file (if Audit Logging is enabled) to see exactly what was sent to the model.

## Auto-Generated Documentation

The project includes a script that regenerates reference documentation from source code:

```bash
python3 tools/generate_docs.py
```

This script parses `AgentSettings.kt`, `SettingsPanel.kt`, `McpToolCatalog.kt`, and `McpTools.kt` to produce:

*   `GitBook/reference/settings-reference.md` — All settings with defaults and ranges.
*   `GitBook/mcp/tools-reference.md` — Quick MCP tool catalog table.
*   `GitBook/mcp/tools-reference-detailed.md` — Detailed MCP tool reference with input schemas.

> **Important**: If you manually edit these three files, your changes will be overwritten the next time `generate_docs.py` runs. Make structural changes in the source code or in the script itself.

## Manual UI Checks
Before a release, perform these sanity checks:
- [ ] Load extension in Burp Community and Pro.
- [ ] Verify tab appearance on macOS (Retina), Linux, and Windows (HiDPI).
- [ ] Check context menus appear on Request/Response and Issues.
- [ ] Verify scrolling in the Chat interface.