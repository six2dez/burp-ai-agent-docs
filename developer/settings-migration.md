# Settings Migration

`AgentSettingsRepository.load()` stamps the Burp preferences store with a schema version and runs forward-only migrations before loading settings. This page explains the model and shows how to add a new migration step.

## Schema Version Field

The schema version is stored in Burp preferences under the key `settings.schema.version` as an integer. `AgentSettingsRepository.CURRENT_SETTINGS_SCHEMA_VERSION` is the authoritative internal target; loads treat an absent value as v1.

## Current Schema

| Version | Changes |
| :--- | :--- |
| v1 | Baseline — all original settings keys. |
| v2 | Normalize `mcp.allowedOrigins` into a clean list; replace the legacy Gemini default command with `--output-format text --model gemini-2.5-flash --yolo`. Introduces MCP proxy history preprocessing keys (`preprocess.proxy.history.*`) with defaults. |
| v3 | Stamp only — no data movement. Reserves the `custom.prompt.library.v1` key for the Custom Prompt Library; missing or malformed JSON loads as an empty library. |
| v4 | Encrypt eight existing plaintext secret preferences in place with `SecretCipher`: the Ollama, LM Studio, OpenAI-compatible, NVIDIA NIM, Perplexity, and Anthropic API keys, plus the MCP token and TLS password. Values already prefixed `ENC1:` and blank values are skipped; plaintext is overwritten only after decrypting the new ciphertext back to the exact original value. |
| v5 | Reserve the encrypted external-MCP-server blob. Existing installs load an empty server list; no old field is moved because the key is new. |

## Migration Flow

```mermaid
flowchart TD
    Load[load(api)]
    Read[Read settings.schema.version]
    V0{version absent?}
    V1{version < 2?}
    V2{version < 3?}
    V3{version < 4?}
    V4{version < 5?}
    M2["migrateToSchemaV2<br/>normalize origins<br/>replace Gemini default"]
    M3["v3 stamp<br/>custom prompt library default"]
    M4["migrateToSchemaV4<br/>encrypt 8 secret keys<br/>round-trip before overwrite"]
    M5["v5 stamp<br/>external MCP servers default empty"]
    WriteStamp[Write schema.version = 5]
    Done[Return AgentSettings]

    Load --> Read --> V0
    V0 -->|Yes| V1
    V0 -->|No| V1
    V1 -->|Yes| M2 --> V2
    V1 -->|No| V2
    V2 -->|Yes| M3 --> V3
    V2 -->|No| V3
    V3 -->|Yes| M4 --> V4
    V3 -->|No| V4
    V4 -->|Yes| M5 --> WriteStamp --> Done
    V4 -->|No| Done
```

Guarantees:

* Migrations are **forward-only**. Most versions add defaults; v4 deliberately replaces plaintext secret values after a verified encryption round trip.
* Migrations are **idempotent at their field boundary**. Re-running the chain on already-v5 preferences is a no-op, and the v4 `ENC1:` guard prevents double encryption.
* A missing new key means "use the default" unless the migration explicitly owns old data that must be transformed.
* Migrations **never read UI state**. They operate only on the preferences snapshot.

## Adding a New Migration Step

1. Bump `CURRENT_SETTINGS_SCHEMA_VERSION` by 1 (for example, from `5` to `6`).
2. Add a new private helper in `AgentSettingsRepository`:

   ```kotlin
   private fun migrateToSchemaV6() {
       // Example: rename a key.
       val legacy = prefs.getString("legacy.key")
       if (legacy != null) {
           prefs.setString("new.key", legacy)
           prefs.deleteString("legacy.key")
       }
   }
   ```

3. Call it from `migrateIfNeeded()`, guarded by `if (effectiveVersion < 6)`, then set `effectiveVersion = 6`. Keep all earlier guards intact so users on v1 still run every step in order.
4. Let the chain stamp `KEY_SETTINGS_SCHEMA_VERSION` after walking the version guards. Do not write the marker inside the helper.
5. Add cases to `AgentSettingsMigrationTest` or the feature-specific migration test (for example `ExternalMcpSettingsMigrationTest`) that start from older versions and assert the final shape.

The version guard decides whether a migration step runs. Inside a step, field-level presence checks and idempotency markers are expected—for example, v4 skips blank or already-`ENC1:` values. Tests should cover the oldest supported source version, immediate predecessor, already-current state, partial data, and a second load.

## Downgrade Policy

Downgrades are not supported. Schema v4 is particularly important: older builds do not understand the `ENC1:` secret representation, and v5 adds a settings blob unknown to them. If a user must roll back:

1. Close Burp.
2. Restore a complete preferences or project backup created by the older version, **or** reset the extension's preferences and re-enter settings in the older build.
3. Reopen Burp with the older JAR and verify backend/MCP credentials before use.

Changing only `settings.schema.version` is not a rollback: it neither decrypts v4 secrets nor converts newer data shapes. Back up the Burp project/preferences before testing migration changes.

## Related Pages

* [Settings Reference](../reference/settings-reference.md)
* [Architecture](architecture.md)
