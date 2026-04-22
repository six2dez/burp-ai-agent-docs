# Testing & Debugging

## Running Tests

The project uses JUnit 5 for testing.

### Fast suite (default in CI PR gate)
```bash
./gradlew test -PexcludeHeavyTests=true
```
Excludes `*IntegrationTest`, `*ConcurrencyTest`, `*BackpressureTest`, `*RestartPolicyTest`. Fast enough to run on every PR across Linux, macOS, and Windows in the matrix defined in `.github/workflows/build.yml`.

### Full suite (nightly + on release tag)
```bash
./gradlew test nightlyRegressionTest
```
Adds the integration, concurrency, backpressure, and supervisor restart-policy suites. Automatically executed by `.github/workflows/nightly-regression.yml` at 03:30 UTC and by `.github/workflows/release.yml` on every version tag.

### Integration / concurrency tests
Some tests require a local Ollama instance or mock server. Ensure your environment is set up before running the heavy suite locally.

## Lint

```bash
./gradlew ktlintFormat   # auto-fix style
./gradlew ktlintCheck    # verify style
```

`ktlintCheck` runs in CI as its own job. Until the baseline is clean it is configured as non-blocking (`continue-on-error: true` in the workflow, `ignoreFailures = true` in `build.gradle.kts`). Flip both off with `-PktlintStrict=true` once the repository passes cleanly.

## Coverage

JaCoCo is wired into the default `test` task:

```bash
./gradlew test jacocoTestReport
```

Reports land at `build/reports/jacoco/test/html/index.html` and `build/reports/jacoco/test/jacocoTestReport.xml`. The Linux leg of the PR-gate matrix uploads the XML report as a CI artifact for later aggregation.

## Building from Source

To build the extension JAR:
```bash
./gradlew clean shadowJar
```

The output JAR is `build/libs/Custom-AI-Agent-<version>.jar`. Use that JAR to install the extension in Burp.

## SBOM

```bash
./gradlew cyclonedxBom
```

Produces `build/reports/sbom/bom.json` (CycloneDX 1.5). The release workflow attaches it to the GitHub Release alongside the JAR and its SHA-256 checksum.

## Debugging

If the extension is behaving unexpectedly, check the logs.

### 1. Burp Extension Output
In Burp Suite:
1.  Go to **Extensions** → **Installed**.
2.  Select **Custom AI Agent**.
3.  Click the **Output** tab.
    *   **Standard Output**: Shows general info and initialization messages.
    *   **Errors**: Shows Java stack traces if the extension crashes.

### 2. Java Logs
The extension uses SLF4J. Logs are typically printed to the Burp stdout.
If you launched Burp from a terminal, check the terminal window for detailed logs.

### 3. Agent Debugging
If a backend is failing:
1.  Check the **Status** indicator in the top bar.
2.  If it says **Crashed**, check **Extensions → Installed → Custom AI Agent → Output/Errors** for the exit code.
3.  Check the `audit.jsonl` file (if Audit Logging is enabled) to see exactly what was sent to the model.

## Manual UI Checks
Before a release, perform these sanity checks:
- [ ] Load extension in Burp Community and Pro.
- [ ] Verify tab appearance on macOS (Retina), Linux, and Windows (HiDPI).
- [ ] Check context menus appear on Request/Response and Issues.
- [ ] Verify scrolling in the Chat interface.


## Recommended Local Test Command

To reduce flakiness and keep CI-like behavior locally, use:

```bash
JAVA_HOME=/Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home ./gradlew test --no-daemon -Dorg.gradle.workers.max=1
```

Additional focus areas covered by the test suite:

- Passive scanner JSON parsing with nested/escaped content.
- Injection point extraction for escaped strings, booleans, and nulls.
- Active payload generation paths (numeric/string/UUID).
- Response analyzer diff and time-based detection boundaries.
- Shared HTTP conversation history trimming and concurrent writes.
