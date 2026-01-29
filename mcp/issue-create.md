# Issue Creation (MCP)

The `issue_create` MCP tool allows AI agents to programmatically create Burp audit issues. This is one of the most important tools for agentic workflows, enabling the AI to document its findings directly in Burp.

## Tool Details

| Property | Value |
| :--- | :--- |
| **Tool name** | `issue_create` |
| **Category** | Issues |
| **Unsafe** | No |
| **Default enabled** | Yes |
| **Pro only** | No |

## Input Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `name` | String | Yes | Issue title (e.g., "SQL Injection in login parameter"). |
| `detail` | String | Yes | Full description with evidence, steps, and technical details. |
| `severity` | String | Yes | One of: `INFORMATION`, `LOW`, `MEDIUM`, `HIGH`. |
| `confidence` | String | Yes | One of: `CERTAIN`, `FIRM`, `TENTATIVE`. |
| `baseUrl` | String | Yes | The URL where the issue was found. |
| `remediation` | String | No | Recommended fix for the vulnerability. |
| `background` | String | No | Background information for the issue. |
| `remediationBackground` | String | No | Background information for remediation. |
| `typicalSeverity` | String | No | Typical severity (defaults to `severity`). |
| `httpRequest` | String | No | Raw HTTP request to attach to the issue. |
| `httpResponseContent` | String | No | Raw HTTP response to attach to the issue. |
| `targetHostname` | String | No | Hostname for the attached request. |
| `targetPort` | Int | No | Port for the attached request (default 443). |
| `usesHttps` | Boolean | No | Whether the attached request uses HTTPS (default true). |

## Best Practices

### When to Create Issues

*   **High confidence**: Only create issues when the AI has strong evidence of a vulnerability. Use `TENTATIVE` confidence for suspected but unconfirmed findings.
*   **After verification**: Ideally, the AI should verify the finding (e.g., by sending test requests via `http1_request`) before creating an issue.
*   **Avoid duplicates**: Check existing issues (via `scanner_issues` on Pro) before creating a new one.

### Issue Formatting

*   **Title prefix**: Use `[AI]` prefix for easy identification (e.g., `[AI] Reflected XSS in search parameter`).
*   **Evidence**: Include specific request/response excerpts, parameter names, and payload details.
*   **Reproduction steps**: Number each step so a human can follow along.
*   **Remediation**: Provide actionable fix recommendations, not generic advice.
*   **Plain text**: Issue fields are sanitized to plain text (Markdown formatting removed) before they are stored in Burp.

## Example Usage

An MCP client (like Claude Desktop) might call the tool like this:

```json
{
  "name": "[AI] SQL Injection in user_id parameter",
  "detail": "The user_id parameter in GET /api/users is vulnerable to error-based SQL injection. Payload: user_id=1' OR '1'='1 produced a MySQL syntax error in the response body: 'You have an error in your SQL syntax near...' This indicates unsanitized user input is being interpolated directly into SQL queries.",
  "severity": "HIGH",
  "confidence": "FIRM",
  "baseUrl": "https://example.com/api/users?user_id=1",
  "remediation": "Use parameterized queries (prepared statements) instead of string concatenation for SQL queries. Apply input validation to reject non-numeric values for user_id."
}
```

## Integration with Workflows

The `issue_create` tool is typically the final step in an agentic workflow:

1.  AI browses proxy history (`proxy_http_history_regex`).
2.  AI identifies suspicious endpoint.
3.  AI sends test requests (`http1_request`) to verify.
4.  AI analyzes responses for vulnerability indicators.
5.  AI creates an issue (`issue_create`) with full evidence.

See [Typical Workflows](../examples/typical-workflows.md) for complete examples.
