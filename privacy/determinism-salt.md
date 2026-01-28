# Determinism & Salt

## Determinism mode
When enabled, context ordering is stable for reproducible prompt bundles. This ensures that the same selection of requests/responses produces the same prompt bundle ordering.

## Host anonymization salt
The **Salt** is a secret string used to generate pseudonyms for hostnames in **STRICT** mode.

*   **Why it matters**: If you use the same Salt across two different projects, the host `bank.com` will always be converted to the same pseudonym (e.g., `host-8f2a3b.local`).
*   **Security Best Practice**: Rotate the Salt when starting a new engagement or project to ensure no correlation exists between different chat sessions or audit logs.
