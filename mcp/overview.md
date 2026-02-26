# MCP Overview

Model Context Protocol (MCP) lets external AI clients use Burp data and actions through a controlled interface.

## What MCP Enables

With MCP enabled, an external AI client can:

* query Burp history,
* run analysis tools,
* send controlled requests,
* create issues programmatically.

This keeps the operator in control while expanding automation options.

## Connection Topology

{% tabs %}
{% tab title="SSE (Default)" %}
Primary transport for MCP clients.

Default endpoint:

```text
http://127.0.0.1:9876/sse
```

For external access, enable TLS and include `Authorization: Bearer <token>`.
{% endtab %}

{% tab title="STDIO Bridge" %}
Optional transport for process-based clients.

Enable **STDIO Bridge** in the **MCP Server** settings tab.

Useful when your client expects stdin/stdout MCP server behavior.
{% endtab %}
{% endtabs %}

## Cloud Client Setup (SSE via stdio bridge)

Some desktop clients expect a stdio MCP process. `supergateway` bridges to Burp SSE.

```json
{
  "mcpServers": {
    "burp-ai-agent": {
      "command": "npx",
      "args": [
        "-y",
        "supergateway",
        "--sse",
        "http://127.0.0.1:9876/sse"
      ]
    }
  }
}
```

If token is required:

```json
{
  "mcpServers": {
    "burp-ai-agent": {
      "command": "npx",
      "args": [
        "-y",
        "supergateway",
        "--sse",
        "http://127.0.0.1:9876/sse",
        "--oauth2Bearer",
        "your-token"
      ]
    }
  }
}
```

## Features

* SSE and optional STDIO transport.
* 53+ tools across Burp workflows.
* Unsafe-tool gating.
* Configurable request limiter and body-size caps.
* Health endpoint (`GET /__mcp/health`).
* Privacy-aware tool output filtering.

![Screenshot: MCP settings](../.gitbook/assets/mcp-settings.png)

## Next Steps

* [Security Model](security-model.md)
* [Tools Reference](tools-reference.md)
* [Issue Creation (MCP)](issue-create.md)
