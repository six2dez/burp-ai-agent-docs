# Settings Migration

`AgentSettings.load(api)` stamps every preferences file with a schema version and runs forward-only migrations on load. This page explains the model and shows how to add a new migration step.

## Schema Version Field

The schema version is stored in Burp preferences under the key `settings.schema.version` as an integer. `AgentSettings.CURRENT_SETTINGS_SCHEMA_VERSION` is the authoritative target; loads treat an absent value as v1.

## Current Schema

| Version | Released in | Changes |
| :--- | :--- | :--- |
| v1 | Initial 0.1.x line | Baseline — all original settings keys. |
| v2 | v0.3.0 | Normalize `mcp.allowedOrigins` into a clean list; replace the legacy Gemini default command with `--output-format text --model gemini-2.5-flash --yolo`. Introduces MCP proxy history preprocessing keys (`preprocess.proxy.history.*`) with defaults. |
| v3 | v0.5.0 | Stamp only — no data movement. Reserves the `custom.prompt.library.v1` key for the Custom Prompt Library; missing or malformed JSON loads as an empty library. |

## Migration Flow

```mermaid
flowchart TD
    Load[load(api)]
    Read[Read settings.schema.version]
    V0{version absent?}
    V1{version < 2?}
    V2{version < 3?}
    M2[migrateToSchemaV2\nnormalize origins\nreplace Gemini default]
    M3[v3 stamp\nKEY_CUSTOM_PROMPT_LIBRARY]
    WriteStamp[Write schema.version = 3]
    Done[Return AgentSettings]

    Load --> Read --> V0
    V0 -->|Yes| V1
    V0 -->|No| V1
    V1 -->|Yes| M2 --> V2
    V1 -->|No| V2
    V2 -->|Yes| M3 --> WriteStamp --> Done
    V2 -->|No| Done
```

Guarantees:

* Migrations are **additive**. A missing key means "use the default" — never treat it as an error.
* Migrations are **idempotent**. Re-running the chain on already-v3 preferences is a no-op.
* Migrations **never read UI state**. They operate only on the preferences snapshot.

## Adding a New Migration Step

1. Bump `CURRENT_SETTINGS_SCHEMA_VERSION` by 1 (e.g., from `3` to `4`).
2. Add a new private helper in `AgentSettings`:

   ```kotlin
   private fun migrateToSchemaV4(prefs: PreferenceStore) {
       // Example: rename a key.
       val legacy = prefs.getString("legacy.key")
       if (legacy != null) {
           prefs.setString("new.key", legacy)
           prefs.deleteString("legacy.key")
       }
   }
   ```

3. Call it from the existing `migrateIfNeeded(prefs)` chain, guarded by `if (storedVersion < 4)`. Keep earlier guards intact so users on v1/v2 still run every step in order.
4. Stamp the version at the end of the chain (`prefs.setInteger(KEY_SCHEMA_VERSION, 4)`).
5. Add a test case in `AgentSettingsMigrationTest` that starts from an older version and asserts the final shape.

Do not add branches that run the migration only when a specific key is present — migrations must survive future schema changes that mutate those keys.

## Downgrade Policy

Downgrades are not supported. If a user needs to roll back:

1. Close Burp.
2. Delete the preferences file (Burp → Settings → Project options / User options → export/reset), or hand-edit `settings.schema.version` back to the target version.
3. Reopen Burp with the older JAR.

Because migrations are forward-only, expect anything added after the target version to fall back to defaults.

## Related Pages

* [Settings Reference](../reference/settings-reference.md)
* [Upgrade & Migration Guide](../reference/upgrade-migration.md)
* [Architecture](architecture.md)
