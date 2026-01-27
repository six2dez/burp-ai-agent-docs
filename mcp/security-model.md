# MCP Security Model

The MCP server in Burp AI Agent is designed with a **"Security by Default"** philosophy. Since the Model Context Protocol allows external applications to interact with Burp, we implement multiple layers of defense.

## 1. Local-Only Binding
By default, the MCP server binds to `127.0.0.1`. This ensures that only processes running on your local machine (like Claude Desktop or a local script) can connect. External network access is blocked unless you explicitly change the bind address (not recommended).

## 2. Token Authentication
Every request to the MCP server must include a secret **Token**.
*   The token is auto-generated on the first run.
*   You can view or rotate the token in **Settings → MCP Server**.
*   Clients must provide this token in the header (for SSE) or as a parameter.

## 3. Tool Gating (Safe vs. Unsafe)
Tools are categorized based on their potential impact:

*   **Safe Tools**: Read-only operations (e.g., `proxy_http_history`, `site_map`). These are enabled by default.
*   **Unsafe Tools**: Operations that can modify state or send network traffic (e.g., `http1_request`, `repeater_tab`, `issue_create`).
    *   **Disabled by default**: You must check the "Enable Unsafe Tools" box in settings.
    *   **Pro-Only**: Some tools (like Scanner-related tools) require Burp Suite Professional.

## 4. Origin & Host Validation
To prevent Cross-Site Request Forgery (CSRF) or Cross-Origin attacks:
*   The server validates the `Host` header.
*   It rejects requests with suspicious `Origin` headers that don't match the allowed configuration.

## 5. Privacy Mode Integration
The MCP server is **Privacy-Aware**.
*   If your Privacy Mode is set to `STRICT`, any data returned by an MCP tool (like a request body) is redacted *before* it is sent to the MCP client.
*   This ensures that even if you connect a cloud-based AI agent via MCP, your sensitive data remains protected according to your policy.

## 6. TLS Encryption (Optional)
For added security, especially if you enable external access, you can enable **TLS**.
*   Supports auto-generated self-signed certificates.
*   Supports custom keystores for enterprise environments.