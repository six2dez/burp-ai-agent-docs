# MCP Tools Reference

## Burp Control
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| proxy_intercept | Yes | No | No | Enables or disables Proxy intercept. |
| task_engine_state | Yes | No | No | Sets Burp's task execution engine to paused or running. |

## Collaborator
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| collaborator_generate | No | Yes | No | Generates a Burp Collaborator payload. |
| collaborator_poll | No | Yes | No | Fetches interactions for a Collaborator secret key. |

## Config
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| project_options_get | No | No | No | Outputs project-level configuration as JSON. |
| project_options_set | Yes | No | No | Sets project-level configuration from JSON. |
| user_options_get | No | No | No | Outputs user-level configuration as JSON. |
| user_options_set | Yes | No | No | Sets user-level configuration from JSON. |

## Editor
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| editor_get | No | No | No | Outputs the contents of the active message editor. |
| editor_set | Yes | No | No | Sets the content of the active message editor. |

## Extension
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| status | No | Yes | No | Returns basic extension and Burp version status. |

## History
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| proxy_history_annotate | Yes | No | No | Adds notes/highlights to proxy history items matching a regex. |
| proxy_http_history | No | Yes | No | Displays items within the proxy HTTP history. |
| proxy_http_history_regex | No | Yes | No | Displays proxy HTTP history items matching a regex. |
| proxy_ws_history | No | Yes | No | Displays items within the proxy WebSocket history. |
| proxy_ws_history_regex | No | Yes | No | Displays WebSocket history items matching a regex. |
| response_body_search | No | Yes | No | Searches response bodies in proxy history using a regex. |

## Requests
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| comparer_send | Yes | No | No | Sends one or more items to Burp Comparer. |
| diff_requests | No | Yes | No | Produces a line diff between two requests. |
| find_reflected | No | Yes | No | Finds reflected parameter values in a response. |
| http1_request | Yes | No | No | Issues an HTTP/1.1 request and returns the response. |
| http2_request | Yes | No | No | Issues an HTTP/2 request and returns the response. |
| insertion_points | No | Yes | No | Lists insertion point offsets for a request. |
| intruder | Yes | No | No | Sends a request to Intruder. |
| intruder_prepare | Yes | No | No | Creates an Intruder tab with explicit insertion points. |
| params_extract | No | Yes | No | Extracts parameters from a request. |
| repeater_tab | Yes | No | No | Creates a new Repeater tab with the specified HTTP request. |
| repeater_tab_with_payload | Yes | No | No | Creates a Repeater tab after applying placeholder replacements. |
| request_parse | No | Yes | No | Parses a raw HTTP request into method, path, headers, parameters, and body. |
| response_parse | No | Yes | No | Parses a raw HTTP response into status, headers, and body. |

## Scanner
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| scan_audit_start | Yes | No | Yes | Starts a Burp Scanner audit. |
| scan_audit_start_mode | Yes | No | Yes | Starts a scanner audit using active or passive mode. |
| scan_audit_start_requests | Yes | No | Yes | Starts an audit and adds HTTP requests. |
| scan_crawl_start | Yes | No | Yes | Starts a Burp Scanner crawl. |
| scan_report | Yes | No | Yes | Generates a scanner report to a path. |
| scan_task_delete | Yes | No | Yes | Deletes a crawl/audit task started via MCP. |
| scan_task_status | No | No | Yes | Gets status for a crawl/audit task. |
| scanner_issues | No | Yes | Yes | Displays scanner issues (Burp Pro only). |

## Scope
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| scope_check | No | Yes | No | Checks whether a URL is in scope. |
| scope_exclude | Yes | No | No | Excludes a URL from scope. |
| scope_include | Yes | No | No | Includes a URL in scope. |

## Site Map
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| site_map | No | Yes | No | Displays items within the Burp site map. |
| site_map_regex | No | Yes | No | Displays site map items matching a regex. |

## Utilities
| Tool | Unsafe | Default enabled | Pro only | Description |
|---|---|---|---|---|
| base64_decode | No | Yes | No | Base64 decodes the input string. |
| base64_encode | No | Yes | No | Base64 encodes the input string. |
| cookie_jar_get | No | Yes | No | Returns cookies from Burp's cookie jar (values redacted unless privacy is OFF). |
| decode_as | No | Yes | No | Decodes base64 content using compression codecs (gzip/deflate/brotli). |
| hash_compute | No | Yes | No | Computes a hash for input text (MD5/SHA1/SHA256/SHA512). |
| jwt_decode | No | Yes | No | Decodes JWT header/payload without verifying the signature. |
| random_string | No | Yes | No | Generates a random string of specified length and character set. |
| url_decode | No | Yes | No | URL decodes the input string. |
| url_encode | No | Yes | No | URL encodes the input string. |

