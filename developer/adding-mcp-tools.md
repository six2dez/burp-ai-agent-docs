# Adding MCP Tools

MCP tools follow a **descriptor + handler** pattern. Each tool is a `ToolHandler` that keeps its descriptor, its input schema and its body together, and a `ToolProvider` contributes a list of handlers to the catalog. `McpToolCatalog` is the merged view of every loaded provider.

## Steps

### 1. Choose the Source Set

Which build a tool ships in is decided by the source set that holds it, not by a flag:

* **Extension-native tool (both builds)** — the handler goes in `src/main/kotlin/com/six2dez/burp/aiagent/mcp/tools/NativeTools.kt` and is added to the `NativeToolProvider` list there. The core registers `NativeToolProvider` in code, never through `META-INF/services`, so no classpath can remove or replace it. There are 8 today: `status`, `issue_create`, `ai_analyze`, `ai_passive_scan`, `ai_findings_recent`, `redact_preview`, `ai_audit_query`, `ai_backends_list`. They are the only tools in the BApp Store build (`./gradlew bappJar`).
* **Generic tool (full build only)** — the handler goes in one of the group files under `src/full/kotlin/com/six2dez/burp/aiagent/mcp/tools/` (`GenericUtilityTools.kt`, `GenericHistoryTools.kt`, `GenericSiteMapTools.kt`, `GenericConfigTools.kt`, `GenericRequestTools.kt`, `GenericScannerTools.kt`) and is added to that file's handler list. `GenericToolProvider` concatenates those lists, and it is registered in `src/full/resources/META-INF/services/com.six2dez.burp.aiagent.spi.ToolProvider`, a file only the full JAR carries. There are 51 today, 59 tools in total in the GitHub full build (`./gradlew fullJar`).

The BApp Store build intentionally omits the generic Montoya-API wrappers because PortSwigger's official Burp MCP Server already provides them. `storeJarContentTest` fails the build if a full-only class ends up in the BApp Store JAR.

### 2. Write the Handler

Define the descriptor next to its body with the `toolHandler` helper. A generic tool from `GenericUtilityTools.kt`:

```kotlin
private val urlEncodeHandler: ToolHandler =
    toolHandler(
        descriptor =
            McpToolDescriptor(
                id = "url_encode",
                title = "URL encode",
                description = "URL encodes the input string.",
                category = "Utilities",
                defaultEnabled = true,
                secTier = SecTier.AUTO,
            ),
        schema = { UrlEncode::class.asInputSchema() },
    ) { args, context ->
        val api = context.api
        val input = decodeToolArgs<UrlEncode>(args)
        api.utilities().urlUtils().encode(input.content)
    }
```

`McpToolDescriptor` has these fields: `id`, `title`, `description`, `category`, `defaultEnabled`, `proOnly` (default `false`; `true` if it requires Burp Pro), `unsafeOnly` (default `false`; `true` if it modifies state or sends traffic), `requiresBurpAi` (default `false`; `true` if the tool sends AI requests, so the executor refuses it while Burp's "Use AI" is off) and `secTier`.

`secTier` has no default: every new tool must make an explicit trust decision. Use `AUTO` only for read-only tools with bounded output, `CONFIRM` when one approval may be remembered for the chat session, and `CONFIRM_EACH` when every invocation needs a fresh decision. This confirmation applies to tool calls emitted by the extension's own model; it is separate from the MCP server's authentication, enablement, unsafe, edition, and scope gates.

Set `pagedArgs = true` on `toolHandler` for a paged tool: `count`, `limit` and `offset` are then defaulted before the body runs.

### 3. Define the Input Model and Schema

Add an `@Serializable` input data class (generic tools keep theirs in `src/full/.../mcp/tools/McpToolModels.kt`; native tools keep theirs in `NativeTools.kt`). The handler's `schema` lambda derives the MCP schema from its Kotlin properties with `asInputSchema()`, and the body decodes the arguments with `decodeToolArgs`.

```kotlin
@Serializable
data class UrlEncode(
    val content: String,
)
```

The current reflection helper emits property types and a `required` list; it does not emit descriptions or Kotlin default values. It marks every non-null property as required in the advertised schema, even when Kotlin deserialization has a default. Runtime decoding still accepts an omitted defaulted property. Keep the user-facing tools reference explicit about both the accepted default and any schema-level mismatch.

### 4. Register the Handler

Add the handler to its provider's list: `NativeToolProvider.handlers()` for a native tool, or the group list a generic file exposes (for example `genericUtilityHandlers`) for a generic tool. `McpToolExecutorImpl.executeToolResult()` finds the handler through the registry and runs its body inside the shared `runTool` path, then passes the result through `McpToolContext.redactIfNeeded`, so the toggle, unsafe and concurrency checks, output limits, telemetry and read-time redaction apply without the handler doing anything. The registry rejects a tool id or alias that another provider already owns, so a plug-in cannot shadow a core tool and inherit its trust tier.

A new plug-in that is not a generic Montoya wrapper implements `com.six2dez.burp.aiagent.spi.ToolProvider` (`handlers()`, and optionally `aliases()`, `onMcpSettingsApplied`, `onMcpServerStopped` and the dynamic-tool hooks) as a class with a public no-arg constructor, and lists it in the same services file under `src/full/resources/META-INF/services/`.

### 5. Add Safety Gating (If Needed)

If your tool is marked as `unsafeOnly`:
* The global **Enable Unsafe Tools** switch (or an explicit per-tool allowance) must permit it before it can run.
* `defaultEnabled` remains an independent catalog choice. For example, a tool may be catalog-enabled yet still blocked by the unsafe master switch.
* Add `McpScopeFilter` enforcement (in `src/full`, beside the generic tools) before any network request or state-changing target operation; the descriptor does not add scope checks automatically.

### 6. Privacy Integration

If your tool returns sensitive data:
* Use `McpToolContext.privacyMode` and `RedactionPolicy.fromMode(...)` for typed decisions.
* Sanitize structured fields **before** serialization. Use the shared helpers where the carrier matches: `sanitizeHeaders`, `sanitizeParameters`, and `maybeAnonymizeUrl` (in `src/full/.../mcp/tools/McpToolHelpers.kt`, for generic tools).
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
* Verify the tool appears exactly once in `McpToolCatalog.all()` with the expected schema, and only in the build it belongs to (`test` for the BApp Store build, `fullTest` for the full build).
* Verify the explicit `SecTier` and the model-emitted approval behavior independently from unsafe gating.
* Verify safety gating: ensure an unsafe tool is blocked when unsafe tools are disabled.
* Verify privacy integration in `STRICT`, `BALANCED`, and `OFF`, including serialized output—not only the pre-serialization object.
* Add one positive removal fixture, one `OFF` byte-preservation control, and a nearby non-secret value to detect destructive over-redaction.
* If the tool introduces a new carrier or producer, add it to the carrier-ownership inventory so a later producer cannot bypass the sanitizer silently.
*   Test edge cases: empty input, invalid parameters, missing Burp Pro features.
