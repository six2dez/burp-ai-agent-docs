# Release Checklist

Use this checklist for release preparation.

1. Bump version.
2. Run `./gradlew clean shadowJar`.
3. Run `./gradlew test`.
4. Run manual UI checks (macOS, Linux, Windows).
5. Verify MCP tools and scanner flows.
6. Verify warnings/safety controls.
7. Update changelog.
8. Publish release artifacts.
