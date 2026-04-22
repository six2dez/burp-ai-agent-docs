# Release Checklist

Use this checklist for release preparation.

## Before tagging

1. Merge all target PRs to `main`.
2. Update `CHANGELOG.md` — move the `[Unreleased]` entries under a new `## [X.Y.Z] - YYYY-MM-DD` heading. The release workflow extracts that section as the GitHub Release body.
3. Bump `version = "X.Y.Z"` in `build.gradle.kts`.
4. Run the full local gate:
   ```bash
   JAVA_HOME=/path/to/jdk-21 ./gradlew clean ktlintCheck test nightlyRegressionTest shadowJar
   JAVA_HOME=/path/to/jdk-21 ./gradlew cyclonedxBom --no-configuration-cache
   ```
   `cyclonedxBom` is split out and runs without the configuration cache because `org.cyclonedx.bom:1.10.0` uses `Task.project` at execution time, which the cache forbids. Upgrading the plugin past 2.x fixes this but pulls in a larger dep graph — track separately.
5. Smoke-test the produced JAR (`build/libs/Custom-AI-Agent-<version>.jar`) in Burp Community and Burp Pro.
6. Manual UI checks on macOS (Retina), Linux, and Windows (HiDPI). Context menus, chat, preview dialog, scanners, MCP toggle.
7. Verify the MCP server accepts requests only with a valid bearer token when external access is enabled.

## Cut the release

1. Commit the version + CHANGELOG changes.
2. Tag: `git tag vX.Y.Z && git push origin vX.Y.Z`.
3. `.github/workflows/release.yml` runs automatically and performs:
   - ktlint check
   - full `test nightlyRegressionTest` suite
   - `shadowJar` + `cyclonedxBom`
   - SHA-256 checksum of the JAR
   - CHANGELOG section extraction for the release body
   - GitHub Release with JAR + SBOM + checksum attached

## After the release

1. Verify the GitHub Release page shows the JAR, `*.jar.sha256`, and `bom.json`.
2. Download the JAR fresh, verify its SHA-256 matches the published checksum, and smoke-load it in Burp.
3. Post the release link to the documentation site and issue tracker.
4. Start a new `[Unreleased]` section in `CHANGELOG.md` for the next cycle.

## Post-release follow-up

- Review Dependabot PRs for the new cycle.
- Review triage of any issues opened against the previous release.
- Update screenshots in `screenshots/` and `.gitbook/assets/` if UI changed meaningfully.
