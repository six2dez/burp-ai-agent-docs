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
    secTier = SecTier.AUTO, // explicit approval tier for model-emitted calls
    nativeTool = false     // see "Store Build vs Full Build" below
)
```

`secTier` has no default: every new tool must make an explicit trust decision. Use `AUTO` only for read-only tools with bounded output, `CONFIRM` when one approval may be remembered for the chat session, and `CONFIRM_EACH` when every invocation needs a fresh decision. This confirmation applies to tool calls emitted by the extension's own model; it is separate from the MCP server's authentication, enablement, unsafe, edition, and scope gates.

#### Store Build vs Full Build

The `nativeTool` flag decides whether a tool ships in the **BApp Store build**. The catalog filters on it:

```kotlin
fun available(storeBuild: Boolean = BuildFlags.STORE_BUILD): List<McpToolDescriptor> =
    if (storeBuild) tools.filter { it.nativeTool } else tools
```

* `nativeTool = true` — the tool is an extension-native AI tool and is registered in **both** builds. The BApp Store build (`./gradlew shadowJar -PstoreBuild=true`) exposes exactly these 8 tools: `status`, `issue_create`, `ai_analyze`, `ai_passive_scan`, `ai_findings_recent`, `redact_preview`, `ai_audit_query`, `ai_backends_list`.
* `nativeTool = false` (default) — the tool is a generic Montoya-API wrapper (proxy history, repeater, scanner, scope, site map, intruder, collaborator, utilities, …). It ships only in the **full build** (`./gradlew shadowJar`, the default GitHub-release artifact), which registers all 59 MCP tools.

`BuildFlags.STORE_BUILD` is a generated compile-time constant set from the `-PstoreBuild` Gradle property, so the same source compiles into either artifact. The store build intentionally omits the generic tools because PortSwigger's official Burp MCP Server already provides those.

### 2. Define the Input Model and Schema

Add an `@Serializable` input data class in `McpToolModels.kt` (or the owning native-tool file). `McpToolExecutor.inputSchema()` maps the tool ID to that class, and `asInputSchema()` derives the MCP schema from its Kotlin properties.

```kotlin
@Serializable
data class MyToolInput(
    val paramName: String,
    val limit: Int = 10,
)
```

```kotlin
"my_tool" -> MyToolInput::class.asInputSchema()
```

The current reflection helper emits property types and a `required` list; it does not emit descriptions or Kotlin default values. It marks every non-null property as required in the advertised schema, even when Kotlin deserialization has a default. Runtime decoding still accepts an omitted defaulted property. Keep the user-facing tools reference explicit about both the accepted default and any schema-level mismatch.

### 3. Implement the Executor Branch

Add the execution branch to `McpToolExecutorImpl.executeToolResult()`. Decode with the production serializer, use the supplied `McpToolContext`, and return text from inside the shared `runTool` path so concurrency, safety, output limits, telemetry, and read-time redaction remain active.

```kotlin
"my_tool" -> {
    val input = decode<MyToolInput>(normalizedArgs)
    val result = api.someOperation(input.paramName, input.limit)
    result.toString()
}
```

Then add the ID to the appropriate list in `McpToolRegistrations` (`McpToolHandlers.kt`). The category wrapper (`RequestTools.kt`, `HistoryTools.kt`, and so on) registers that list with the server. A catalog entry without this registration is visible in settings but is not callable; a registered ID missing from the catalog is skipped.

### 4. Add Safety Gating (If Needed)

If your tool is marked as `unsafeOnly`:
* The global **Enable Unsafe Tools** switch (or an explicit per-tool allowance) must permit it before it can run.
* `defaultEnabled` remains an independent catalog choice. For example, a tool may be catalog-enabled yet still blocked by the unsafe master switch.
* Add `McpScopeFilter` enforcement before any network request or state-changing target operation; the descriptor does not add scope checks automatically.

### 5. Privacy Integration

If your tool returns sensitive data:
* Use `McpToolContext.privacyMode` and `RedactionPolicy.fromMode(...)` for typed decisions.
* Sanitize structured fields **before** serialization. Use the shared helpers where the carrier matches: `sanitizeHeaders`, `sanitizeParameters`, and `maybeAnonymizeUrl`.
* Keep the final result on the normal executor path so `McpToolContext.redactIfNeeded` also applies the current read-time text policy.
* Do not rely on the generic text pass to infer a secret stored under keys such as `value`, `detail`, or `url`; it no longer has the producer's type information.
* Example: `cookie_jar_get` redacts cookie values unless privacy mode is `OFF`, while `request_parse` strips parameters identified by Montoya as `COOKIE` before producing JSON.

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
* Verify catalog/registration/schema parity so the ID appears once in `McpToolCatalog`, once in the correct `McpToolRegistrations` group, and in `inputSchema()`.
* Verify the explicit `SecTier` and the model-emitted approval behavior independently from unsafe gating.
* Verify safety gating: ensure an unsafe tool is blocked when unsafe tools are disabled.
* Verify privacy integration in `STRICT`, `BALANCED`, and `OFF`, including serialized output—not only the pre-serialization object.
* Add one positive removal fixture, one `OFF` byte-preservation control, and a nearby non-secret value to detect destructive over-redaction.
* If the tool introduces a new carrier or producer, add it to the carrier-ownership inventory so a later producer cannot bypass the sanitizer silently.
*   Test edge cases: empty input, invalid parameters, missing Burp Pro features.
