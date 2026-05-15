# MCP Tools Reference

Canonical reference for every MCP tool exposed by the extension. Each category has a summary table (safety, default exposure, Pro-only flag, one-line description) followed by per-tool input schemas.

```mermaid
mindmap
  root((MCP Tools))
    Burp Control
    Collaborator
    Config
    Editor
    Issues
    Extension
    History
    Requests
    Scanner
    Scope
    Site Map
    Utilities
```

Conventions:

* **Unsafe = Yes** means the tool can mutate Burp state or send traffic to targets. Tools marked unsafe are gated behind the **Enable Unsafe Tools** master switch in **Settings → MCP Server**.
* **Default enabled = Yes** means the tool is available in agent profiles without an explicit opt-in.
* **Pro only = Yes** means the tool requires Burp Suite Professional. Community-edition clients see it return an error if invoked.
* **Input fields: none** means the tool takes no parameters.

## Burp Control

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `proxy_intercept` | Yes | No | No | Enables or disables Proxy intercept. |
| `task_engine_state` | Yes | No | No | Sets Burp's task execution engine to paused or running. |

### proxy_intercept

| Name | Type | Required | Default |
|---|---|---|---|
| `intercepting` | Boolean | Yes | — |

### task_engine_state

| Name | Type | Required | Default |
|---|---|---|---|
| `running` | Boolean | Yes | — |

## Collaborator

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `collaborator_generate` | No | Yes | No | Generates a Burp Collaborator payload. |
| `collaborator_poll` | No | Yes | No | Fetches interactions for a Collaborator secret key. |

### collaborator_generate

| Name | Type | Required | Default |
|---|---|---|---|
| `customData` | String? | No | `null` |
| `options` | List\<String> | No | `emptyList()` |

### collaborator_poll

| Name | Type | Required | Default |
|---|---|---|---|
| `secretKey` | String | Yes | — |
| `includeHttp` | Boolean | No | `false` |

## Config

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `project_options_get` | No | No | No | Outputs project-level configuration as JSON. |
| `project_options_set` | Yes | No | No | Sets project-level configuration from JSON. |
| `user_options_get` | No | No | No | Outputs user-level configuration as JSON. |
| `user_options_set` | Yes | No | No | Sets user-level configuration from JSON. |

### project_options_get

Input fields: none.

### project_options_set

| Name | Type | Required | Default |
|---|---|---|---|
| `json` | String | Yes | — |

### user_options_get

Input fields: none.

### user_options_set

| Name | Type | Required | Default |
|---|---|---|---|
| `json` | String | Yes | — |

## Editor

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `editor_get` | No | No | No | Outputs the contents of the active message editor. |
| `editor_set` | Yes | No | No | Sets the content of the active message editor. |

### editor_get

Input fields: none.

### editor_set

| Name | Type | Required | Default |
|---|---|---|---|
| `text` | String | Yes | — |

## Issues

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `issue_create` | No | Yes | No | Creates a custom audit issue in Burp's issue list. |

### issue_create

| Name | Type | Required | Default |
|---|---|---|---|
| `name` | String | Yes | — |
| `detail` | String | Yes | — |
| `baseUrl` | String | Yes | — |
| `severity` | String | Yes | — |
| `confidence` | String | Yes | — |
| `remediation` | String? | No | `null` |
| `background` | String? | No | `null` |
| `remediationBackground` | String? | No | `null` |
| `typicalSeverity` | String? | No | `null` |
| `httpRequest` | String? | No | `null` |
| `httpResponseContent` | String? | No | `null` |
| `targetHostname` | String | No | `""` |
| `targetPort` | Int | No | `443` |
| `usesHttps` | Boolean | No | `true` |

See [Issue Creation (MCP)](issue-create.md) for guidance on building well-formed issue payloads.

## Extension

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `status` | No | Yes | No | Returns basic extension and Burp status. |

### status

Input fields: none.

## History

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `proxy_history_annotate` | Yes | No | No | Adds notes/highlights to proxy history items matching a regex. |
| `proxy_http_history` | No | Yes | No | Displays items within the proxy HTTP history. |
| `proxy_http_history_regex` | No | Yes | No | Displays proxy HTTP history items matching a regex. |
| `proxy_ws_history` | No | Yes | No | Displays items within the proxy WebSocket history. |
| `proxy_ws_history_regex` | No | Yes | No | Displays WebSocket history items matching a regex. |
| `response_body_search` | No | Yes | No | Searches response bodies in proxy history using a regex. |

History tools that surface proxy data flow through the MCP proxy-history preprocessor (see **Settings → MCP Server → MCP Proxy History Preprocessing**).

### proxy_history_annotate

| Name | Type | Required | Default |
|---|---|---|---|
| `regex` | String | Yes | — |
| `note` | String | Yes | — |
| `highlight` | String? | No | `null` |
| `scopeOnly` | Boolean | No | `true` |
| `limit` | Int | No | `20` |

### proxy_http_history

| Name | Type | Required | Default |
|---|---|---|---|
| `count` | Int | Yes | — |
| `offset` | Int | Yes | — |

### proxy_http_history_regex

| Name | Type | Required | Default |
|---|---|---|---|
| `regex` | String | Yes | — |
| `count` | Int | Yes | — |
| `offset` | Int | Yes | — |

### proxy_ws_history

| Name | Type | Required | Default |
|---|---|---|---|
| `count` | Int | Yes | — |
| `offset` | Int | Yes | — |

### proxy_ws_history_regex

| Name | Type | Required | Default |
|---|---|---|---|
| `regex` | String | Yes | — |
| `count` | Int | Yes | — |
| `offset` | Int | Yes | — |

### response_body_search

| Name | Type | Required | Default |
|---|---|---|---|
| `regex` | String | Yes | — |
| `count` | Int | No | `5` |
| `offset` | Int | No | `0` |
| `scopeOnly` | Boolean | No | `true` |

## Requests

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `comparer_send` | Yes | No | No | Sends one or more items to Burp Comparer. |
| `diff_requests` | No | Yes | No | Produces a line diff between two requests. |
| `find_reflected` | No | Yes | No | Finds reflected parameter values in a response. |
| `http1_request` | Yes | No | No | Issues an HTTP/1.1 request and returns the response. Optional in agent profiles. |
| `http2_request` | Yes | No | No | Issues an HTTP/2 request and returns the response. Optional in agent profiles. |
| `insertion_points` | No | Yes | No | Lists insertion point offsets for a request. |
| `intruder` | Yes | No | No | Sends a request to Intruder. |
| `intruder_prepare` | Yes | No | No | Creates an Intruder tab with explicit insertion points. |
| `params_extract` | No | Yes | No | Extracts parameters from a request. |
| `repeater_tab` | Yes | No | No | Creates a new Repeater tab with the specified HTTP request. |
| `repeater_tab_with_payload` | Yes | No | No | Creates a Repeater tab after applying placeholder replacements. |
| `request_parse` | No | Yes | No | Parses a raw HTTP request into method, path, headers, parameters, and body. |
| `response_parse` | No | Yes | No | Parses a raw HTTP response into status, headers, and body. |

{% hint style="info" %}
`http1_request` and `http2_request` require **Enable Unsafe Tools** in the MCP Server tab. The built-in agent profiles (pentester, bughunter, auditor) list these tools as optional — no validation warning is shown when they are disabled. Custom profiles that explicitly reference these tools will warn until Unsafe Tools are enabled.
{% endhint %}

### comparer_send

| Name | Type | Required | Default |
|---|---|---|---|
| `items` | List\<String> | Yes | — |

### diff_requests

| Name | Type | Required | Default |
|---|---|---|---|
| `requestA` | String | Yes | — |
| `requestB` | String | Yes | — |

### find_reflected

| Name | Type | Required | Default |
|---|---|---|---|
| `request` | String | Yes | — |
| `response` | String | Yes | — |

### http1_request

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |
| `targetHostname` | String | Yes | — |
| `targetPort` | Int | Yes | — |
| `usesHttps` | Boolean | Yes | — |

### http2_request

| Name | Type | Required | Default |
|---|---|---|---|
| `pseudoHeaders` | Map\<String, String> | Yes | — |
| `headers` | Map\<String, String> | Yes | — |
| `requestBody` | String | Yes | — |
| `targetHostname` | String | Yes | — |
| `targetPort` | Int | Yes | — |
| `usesHttps` | Boolean | Yes | — |

### insertion_points

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |
| `mode` | String | No | `"REPLACE_BASE_PARAMETER_VALUE_WITH_OFFSETS"` |

### intruder

| Name | Type | Required | Default |
|---|---|---|---|
| `tabName` | String? | Yes | — |
| `content` | String | Yes | — |
| `targetHostname` | String | Yes | — |
| `targetPort` | Int | Yes | — |
| `usesHttps` | Boolean | Yes | — |

### intruder_prepare

| Name | Type | Required | Default |
|---|---|---|---|
| `tabName` | String? | Yes | — |
| `content` | String | Yes | — |
| `insertionPoints` | List\<InsertionPointRange> | No | `emptyList()` |
| `mode` | String | No | `"REPLACE_BASE_PARAMETER_VALUE_WITH_OFFSETS"` |
| `targetHostname` | String | Yes | — |
| `targetPort` | Int | Yes | — |
| `usesHttps` | Boolean | Yes | — |

### params_extract

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |

### repeater_tab

| Name | Type | Required | Default |
|---|---|---|---|
| `tabName` | String? | Yes | — |
| `content` | String | Yes | — |
| `targetHostname` | String | Yes | — |
| `targetPort` | Int | Yes | — |
| `usesHttps` | Boolean | Yes | — |

### repeater_tab_with_payload

| Name | Type | Required | Default |
|---|---|---|---|
| `tabName` | String? | Yes | — |
| `content` | String | Yes | — |
| `replacements` | Map\<String, String> | Yes | — |
| `targetHostname` | String | Yes | — |
| `targetPort` | Int | Yes | — |
| `usesHttps` | Boolean | Yes | — |

### request_parse

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |
| `includeBody` | Boolean | No | `false` |

### response_parse

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |
| `includeBody` | Boolean | No | `false` |

## Scanner

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `scan_audit_start` | Yes | No | Yes | Starts a Burp Scanner audit. |
| `scan_audit_start_mode` | Yes | No | Yes | Starts a scanner audit using active or passive mode. |
| `scan_audit_start_requests` | Yes | No | Yes | Starts an audit and adds HTTP requests. |
| `scan_crawl_start` | Yes | No | Yes | Starts a Burp Scanner crawl. |
| `scan_report` | Yes | No | Yes | Generates a scanner report to a path. |
| `scan_task_delete` | Yes | No | Yes | Deletes a crawl/audit task started via MCP. |
| `scan_task_status` | No | No | Yes | Gets status for a crawl/audit task. |
| `scanner_issues` | No | Yes | Yes | Displays scanner issues (Burp Pro only). |

### scan_audit_start

| Name | Type | Required | Default |
|---|---|---|---|
| `builtInConfiguration` | String | Yes | — |

### scan_audit_start_mode

| Name | Type | Required | Default |
|---|---|---|---|
| `mode` | String | Yes | — |
| `requests` | List\<String> | No | `emptyList()` |
| `targetHostname` | String | No | `""` |
| `targetPort` | Int | No | `0` |
| `usesHttps` | Boolean | No | `true` |

### scan_audit_start_requests

| Name | Type | Required | Default |
|---|---|---|---|
| `builtInConfiguration` | String | Yes | — |
| `requests` | List\<String> | Yes | — |
| `targetHostname` | String | Yes | — |
| `targetPort` | Int | Yes | — |
| `usesHttps` | Boolean | Yes | — |

### scan_crawl_start

| Name | Type | Required | Default |
|---|---|---|---|
| `seedUrls` | List\<String> | Yes | — |

### scan_report

| Name | Type | Required | Default |
|---|---|---|---|
| `taskId` | String? | Yes | — |
| `allIssues` | Boolean | Yes | — |
| `format` | String | Yes | — |
| `path` | String | Yes | — |

### scan_task_delete

| Name | Type | Required | Default |
|---|---|---|---|
| `taskId` | String | Yes | — |

### scan_task_status

| Name | Type | Required | Default |
|---|---|---|---|
| `taskId` | String | Yes | — |

### scanner_issues

| Name | Type | Required | Default |
|---|---|---|---|
| `count` | Int | Yes | — |
| `offset` | Int | Yes | — |

## Scope

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `scope_check` | No | Yes | No | Checks whether a URL is in scope. |
| `scope_exclude` | Yes | No | No | Excludes a URL from scope. |
| `scope_include` | Yes | No | No | Includes a URL in scope. |

### scope_check

| Name | Type | Required | Default |
|---|---|---|---|
| `url` | String | Yes | — |

### scope_exclude

| Name | Type | Required | Default |
|---|---|---|---|
| `url` | String | Yes | — |

### scope_include

| Name | Type | Required | Default |
|---|---|---|---|
| `url` | String | Yes | — |

## Site Map

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `site_map` | No | Yes | No | Displays items within the Burp site map. |
| `site_map_regex` | No | Yes | No | Displays site map items matching a regex. |

### site_map

| Name | Type | Required | Default |
|---|---|---|---|
| `count` | Int | Yes | — |
| `offset` | Int | Yes | — |

### site_map_regex

| Name | Type | Required | Default |
|---|---|---|---|
| `regex` | String | Yes | — |
| `count` | Int | Yes | — |
| `offset` | Int | Yes | — |

## Utilities

| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| `base64_decode` | No | Yes | No | Base64 decodes the input string. |
| `base64_encode` | No | Yes | No | Base64 encodes the input string. |
| `cookie_jar_get` | No | Yes | No | Returns cookies from Burp's cookie jar (values redacted unless privacy is OFF). |
| `decode_as` | No | Yes | No | Decodes base64 content using compression codecs (gzip/deflate/brotli). |
| `hash_compute` | No | Yes | No | Computes a hash for input text (MD5/SHA1/SHA256/SHA512). |
| `jwt_decode` | No | Yes | No | Decodes JWT header/payload without verifying the signature. |
| `random_string` | No | Yes | No | Generates a random string of specified length and character set. |
| `url_decode` | No | Yes | No | URL decodes the input string. |
| `url_encode` | No | Yes | No | URL encodes the input string. |

### base64_decode

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |

### base64_encode

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |

### cookie_jar_get

| Name | Type | Required | Default |
|---|---|---|---|
| `domain` | String? | No | `null` |
| `includeSubdomains` | Boolean | No | `true` |
| `scopeOnly` | Boolean | No | `true` |
| `includeValues` | Boolean | No | `false` |

### decode_as

| Name | Type | Required | Default |
|---|---|---|---|
| `base64` | String | Yes | — |
| `encoding` | String | Yes | — |

### hash_compute

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |
| `algorithm` | String | Yes | — |

### jwt_decode

| Name | Type | Required | Default |
|---|---|---|---|
| `token` | String | Yes | — |

### random_string

| Name | Type | Required | Default |
|---|---|---|---|
| `length` | Int | Yes | — |
| `characterSet` | String | Yes | — |

### url_decode

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |

### url_encode

| Name | Type | Required | Default |
|---|---|---|---|
| `content` | String | Yes | — |

## Related Pages

* [MCP Overview](overview.md)
* [Security Model](security-model.md)
* [Issue Creation (MCP)](issue-create.md)
