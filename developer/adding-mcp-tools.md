# Adding MCP Tools

MCP tools follow a **descriptor + handler** pattern. Each tool has metadata in the catalog and an implementation in the handler.

## Steps

### 1. Add Tool Descriptor in `McpToolCatalog`

Define the tool's metadata:

```kotlin
McpToolDescriptor(
    id = "my_tool",
    title = "My Tool Title",
    description = "What this tool does.",
    category = "MyCategory",
    defaultEnabled = true, // whether it's enabled by default
    proOnly = false,       // true if it requires Burp Pro
    unsafeOnly = false,    // true if it modifies state or sends traffic
    nativeTool = false     // see "Store Build vs Full Build" below
)
```

#### Store Build vs Full Build

The `nativeTool` flag decides whether a tool ships in the **BApp Store build**. The catalog filters on it:

```kotlin
fun available(storeBuild: Boolean = BuildFlags.STORE_BUILD): List<McpToolDescriptor> =
    if (storeBuild) tools.filter { it.nativeTool } else tools
```

* `nativeTool = true` — the tool is an extension-native AI tool and is registered in **both** builds. The BApp Store build (`./gradlew shadowJar -PstoreBuild=true`) exposes exactly these 8 tools: `status`, `issue_create`, `ai_analyze`, `ai_passive_scan`, `ai_findings_recent`, `redact_preview`, `ai_audit_query`, `ai_backends_list`.
* `nativeTool = false` (default) — the tool is a generic Montoya-API wrapper (proxy history, repeater, scanner, scope, site map, intruder, collaborator, utilities, …). It ships only in the **full build** (`./gradlew shadowJar`, the default GitHub-release artifact), which registers all 59 MCP tools.

`BuildFlags.STORE_BUILD` is a generated compile-time constant set from the `-PstoreBuild` Gradle property, so the same source compiles into either artifact. The store build intentionally omits the generic tools because PortSwigger's official Burp MCP Server already provides those.

### 2. Implement Handler in `McpTools`

Add a handler function that:
*   Receives the tool input (parsed from JSON).
*   Accesses Burp APIs via `McpToolContext`.
*   Returns a result string or structured response.

```kotlin
"my_tool" -> {
    val param = input.getString("paramName")
    // Use ctx.api to access Burp APIs
    val result = ctx.api.someOperation(param)
    McpToolResult.text(result)
}
```

### 3. Add Input Schema

Define the JSON Schema for tool input parameters. This tells MCP clients what parameters the tool accepts:

```kotlin
"my_tool" to buildJsonObject {
    put("type", "object")
    putJsonObject("properties") {
        putJsonObject("paramName") {
            put("type", "string")
            put("description", "Description of this parameter")
        }
    }
    putJsonArray("required") { add("paramName") }
}
```

### 4. Add Safety Gating (If Needed)

If your tool is marked as `unsafeOnly`:
*   It will be disabled by default.
*   Users must enable "Unsafe Tools" in settings to use it.
*   Consider adding scope checks to prevent operations on out-of-scope targets.

### 5. Privacy Integration

If your tool returns sensitive data:
*   Use the `McpToolContext.privacyMode` to check the active privacy level.
*   Apply redaction through the context's redaction pipeline before returning results.
*   Example: The `cookie_jar_get` tool redacts cookie values unless privacy mode is OFF.

## Tool Categories

Existing categories for reference:
*   **Burp Control**: Proxy intercept, task engine state.
*   **Collaborator**: Payload generation and polling.
*   **Config**: Project/user options.
*   **Editor**: Message editor get/set.
*   **Extension**: Status information.
*   **History**: Proxy HTTP/WebSocket history.
*   **Issues**: Issue creation.
*   **Requests**: HTTP requests, Repeater, Intruder, parsing.
*   **Scanner**: Audit, crawl, reports (Pro only).
*   **Scope**: URL scope management.
*   **Site Map**: Site map browsing and search.
*   **Utilities**: Encoding, hashing, JWT, compression.

## Testing

*   Test your tool handler with mock `McpToolContext` objects.
*   Verify safety gating: ensure the tool is blocked when unsafe tools are disabled.
*   Verify privacy integration: ensure data is redacted appropriately in STRICT and BALANCED modes.
*   Test edge cases: empty input, invalid parameters, missing Burp Pro features.
