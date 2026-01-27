# Release Checklist

- Bump version.
- Run `./gradlew clean shadowJar`.
- Run `./gradlew test`.
- Manual UI check in Burp (macOS, Linux, Windows).
- Verify MCP tools work.
- Verify scanner toggles and warnings.
- Update changelog.
- Publish release.

