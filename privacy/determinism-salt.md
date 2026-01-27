# Determinism & Salt

## Determinism mode
When enabled, context ordering is stable for reproducible prompt bundles. This ensures that the same selection of requests/responses always results in the same prompt being sent to the AI.

## Host anonymization salt
The **Salt** is a secret string used to generate pseudonyms for hostnames in **STRICT** mode.

*   **Why it matters**: If you use the same Salt across two different projects, the host `bank.com` will always be converted to the same pseudonym (e.g., `anon-target-42.com`). 
*   **Security Best Practice**: Rotate the Salt when starting a new engagement or project to ensure no correlation exists between different chat sessions or audit logs.

![Screenshot: Determinism & salt](../assets/screenshots/determinism-salt.png)