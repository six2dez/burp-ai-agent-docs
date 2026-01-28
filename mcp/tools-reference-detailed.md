# MCP Tools Detailed Reference

## Burp Control
### proxy_intercept
- **Title**: Set proxy intercept state
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Enables or disables Proxy intercept.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### task_engine_state
- **Title**: Set task execution engine state
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Sets Burp's task execution engine to paused or running.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|


## Collaborator
### collaborator_generate
- **Title**: Generate Collaborator payload
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Generates a Burp Collaborator payload.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| customData | String? | No | null |
| options | List<String> | No | emptyList() |

### collaborator_poll
- **Title**: Poll Collaborator interactions
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Fetches interactions for a Collaborator secret key.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| secretKey | String | Yes | - |
| includeHttp | Boolean | No | false |


## Config
### project_options_get
- **Title**: Output project options
- **Unsafe**: No
- **Default enabled**: No
- **Pro only**: No
- **Description**: Outputs project-level configuration as JSON.
- **Input fields**: none

### project_options_set
- **Title**: Set project options
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Sets project-level configuration from JSON.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### user_options_get
- **Title**: Output user options
- **Unsafe**: No
- **Default enabled**: No
- **Pro only**: No
- **Description**: Outputs user-level configuration as JSON.
- **Input fields**: none

### user_options_set
- **Title**: Set user options
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Sets user-level configuration from JSON.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|


## Editor
### editor_get
- **Title**: Get active editor contents
- **Unsafe**: No
- **Default enabled**: No
- **Pro only**: No
- **Description**: Outputs the contents of the active message editor.
- **Input fields**: none

### editor_set
- **Title**: Set active editor contents
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Sets the content of the active message editor.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|


## Issues
### issue_create
- **Title**: Create audit issue
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Creates a custom audit issue in Burp's issue list for AI-discovered findings.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| name | String | Yes | - |
| detail | String | Yes | - |
| baseUrl | String | Yes | - |
| severity | String | Yes | - |
| confidence | String | Yes | - |
| remediation | String? | No | null |
| background | String? | No | null |
| remediationBackground | String? | No | null |
| typicalSeverity | String? | No | null |
| httpRequest | String? | No | null |
| httpResponseContent | String? | No | null |
| targetHostname | String | No | "" |
| targetPort | Int | No | 443 |
| usesHttps | Boolean | No | true |


## Extension
### status
- **Title**: Extension status
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Returns basic extension and Burp version status.
- **Input fields**: none


## History
### proxy_history_annotate
- **Title**: Annotate proxy history
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Adds notes/highlights to proxy history items matching a regex.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| regex | String | Yes | - |
| note | String | Yes | - |
| highlight | String? | No | null |
| scopeOnly | Boolean | No | true |
| limit | Int | No | 20 |

### proxy_http_history
- **Title**: Proxy HTTP history
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Displays items within the proxy HTTP history.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### proxy_http_history_regex
- **Title**: Proxy HTTP history (regex)
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Displays proxy HTTP history items matching a regex.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### proxy_ws_history
- **Title**: Proxy WebSocket history
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Displays items within the proxy WebSocket history.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### proxy_ws_history_regex
- **Title**: Proxy WebSocket history (regex)
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Displays WebSocket history items matching a regex.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### response_body_search
- **Title**: Search response bodies
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Searches response bodies in proxy history using a regex.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| regex | String | Yes | - |
| count | Int | No | 5 |
| offset | Int | No | 0 |
| scopeOnly | Boolean | No | true |


## Requests
### comparer_send
- **Title**: Send to Comparer
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Sends one or more items to Burp Comparer.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### diff_requests
- **Title**: Diff requests
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Produces a line diff between two requests.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### find_reflected
- **Title**: Find reflected values
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Finds reflected parameter values in a response.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### http1_request
- **Title**: Send HTTP/1.1 request
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Issues an HTTP/1.1 request and returns the response.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| content | String | Yes | - |
| targetHostname | String | Yes | - |
| targetPort | Int | Yes | - |
| usesHttps | Boolean | Yes | - |

### http2_request
- **Title**: Send HTTP/2 request
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Issues an HTTP/2 request and returns the response.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| pseudoHeaders | Map<String, String> | Yes | - |
| headers | Map<String, String> | Yes | - |
| requestBody | String | Yes | - |
| targetHostname | String | Yes | - |
| targetPort | Int | Yes | - |
| usesHttps | Boolean | Yes | - |

### insertion_points
- **Title**: List insertion points
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Lists insertion point offsets for a request.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### intruder
- **Title**: Send to Intruder
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Sends a request to Intruder.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| tabName | String? | Yes | - |
| content | String | Yes | - |
| targetHostname | String | Yes | - |
| targetPort | Int | Yes | - |
| usesHttps | Boolean | Yes | - |

### intruder_prepare
- **Title**: Prepare Intruder tab
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Creates an Intruder tab with explicit insertion points.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| tabName | String? | Yes | - |
| content | String | Yes | - |
| insertionPoints | List<InsertionPointRange> | No | emptyList() |
| mode | String | No | "REPLACE_BASE_PARAMETER_VALUE_WITH_OFFSETS" |
| targetHostname | String | Yes | - |
| targetPort | Int | Yes | - |
| usesHttps | Boolean | Yes | - |

### params_extract
- **Title**: Extract parameters
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Extracts parameters from a request.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### repeater_tab
- **Title**: Create repeater tab
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Creates a new Repeater tab with the specified HTTP request.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| tabName | String? | Yes | - |
| content | String | Yes | - |
| targetHostname | String | Yes | - |
| targetPort | Int | Yes | - |
| usesHttps | Boolean | Yes | - |

### repeater_tab_with_payload
- **Title**: Create repeater tab with payload
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Creates a Repeater tab after applying placeholder replacements.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| tabName | String? | Yes | - |
| content | String | Yes | - |
| replacements | Map<String, String> | Yes | - |
| targetHostname | String | Yes | - |
| targetPort | Int | Yes | - |
| usesHttps | Boolean | Yes | - |

### request_parse
- **Title**: Parse request
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Parses a raw HTTP request into method, path, headers, parameters, and body.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### response_parse
- **Title**: Parse response
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Parses a raw HTTP response into status, headers, and body.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|


## Scanner
### scan_audit_start
- **Title**: Start scanner audit
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: Yes
- **Description**: Starts a Burp Scanner audit.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### scan_audit_start_mode
- **Title**: Start scanner audit (mode)
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: Yes
- **Description**: Starts a scanner audit using active or passive mode.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| mode | String | Yes | - |
| requests | List<String> | No | emptyList() |
| targetHostname | String | No | "" |
| targetPort | Int | No | 0 |
| usesHttps | Boolean | No | true |

### scan_audit_start_requests
- **Title**: Start audit with requests
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: Yes
- **Description**: Starts an audit and adds HTTP requests.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| builtInConfiguration | String | Yes | - |
| requests | List<String> | Yes | - |
| targetHostname | String | Yes | - |
| targetPort | Int | Yes | - |
| usesHttps | Boolean | Yes | - |

### scan_crawl_start
- **Title**: Start crawl
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: Yes
- **Description**: Starts a Burp Scanner crawl.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### scan_report
- **Title**: Generate scanner report
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: Yes
- **Description**: Generates a scanner report to a path.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| taskId | String? | Yes | - |
| allIssues | Boolean | Yes | - |
| format | String | Yes | - |
| path | String | Yes | - |

### scan_task_delete
- **Title**: Delete scan task
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: Yes
- **Description**: Deletes a crawl/audit task started via MCP.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### scan_task_status
- **Title**: Get scan task status
- **Unsafe**: No
- **Default enabled**: No
- **Pro only**: Yes
- **Description**: Gets status for a crawl/audit task.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### scanner_issues
- **Title**: Scanner issues
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: Yes
- **Description**: Displays scanner issues (Burp Pro only).
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|


## Scope
### scope_check
- **Title**: Scope check
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Checks whether a URL is in scope.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### scope_exclude
- **Title**: Exclude from scope
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Excludes a URL from scope.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### scope_include
- **Title**: Include in scope
- **Unsafe**: Yes
- **Default enabled**: No
- **Pro only**: No
- **Description**: Includes a URL in scope.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|


## Site Map
### site_map
- **Title**: Site map
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Displays items within the Burp site map.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### site_map_regex
- **Title**: Site map (regex)
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Displays site map items matching a regex.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|


## Utilities
### base64_decode
- **Title**: Base64 decode
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Base64 decodes the input string.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### base64_encode
- **Title**: Base64 encode
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Base64 encodes the input string.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### cookie_jar_get
- **Title**: Cookie jar
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Returns cookies from Burp's cookie jar (values redacted unless privacy is OFF).
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|
| domain | String? | No | null |
| includeSubdomains | Boolean | No | true |
| scopeOnly | Boolean | No | true |
| includeValues | Boolean | No | false |

### decode_as
- **Title**: Decode content
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Decodes base64 content using compression codecs (gzip/deflate/brotli).
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### hash_compute
- **Title**: Compute hash
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Computes a hash for input text (MD5/SHA1/SHA256/SHA512).
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### jwt_decode
- **Title**: Decode JWT
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Decodes JWT header/payload without verifying the signature.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### random_string
- **Title**: Generate random string
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: Generates a random string of specified length and character set.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### url_decode
- **Title**: URL decode
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: URL decodes the input string.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

### url_encode
- **Title**: URL encode
- **Unsafe**: No
- **Default enabled**: Yes
- **Pro only**: No
- **Description**: URL encodes the input string.
- **Input fields**:

| Name | Type | Required | Default |
|---|---|---|---|

