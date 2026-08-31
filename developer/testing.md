# Testing & Debugging

## Running Tests

The project uses JUnit Platform with JUnit Jupiter `6.0.3` and a Java 21 toolchain.

### Fast suite (default in CI PR gate)
```bash
./gradlew test -PexcludeHeavyTests=true
```
Excludes `*IntegrationTest`, `*ConcurrencyTest`, `*BackpressureTest`, `*RestartPolicyTest`, and `*SupervisionTest`. It runs on every PR across Linux, macOS, and Windows in `.github/workflows/build.yml`.

### Nightly-equivalent local suite
```bash
./gradlew test nightlyRegressionTest edtGuardWithoutAssertionsTest
```
This adds integration, concurrency, backpressure, restart-policy, and supervision suites, then proves the MCP executor's EDT guard with JVM assertions disabled. The nightly workflow runs all three tasks and also builds `shadowJar`. The release workflow runs `test`, `nightlyRegressionTest`, lint, and the artifact build on each version tag; it does not invoke the separate assertion-disabled EDT task.

### Integration / concurrency tests
Heavy suites use local test servers and concurrency fixtures; backend-specific manual checks can additionally require an installed CLI or local model. Read a failing test's setup before assuming external services are required.

## Lint

```bash
./gradlew ktlintFormat   # auto-fix style
./gradlew ktlintCheck    # verify style
```

`ktlintCheck` and `detekt` are blocking CI jobs. Ktlint is strict by default; `-PktlintLenient=true` is an explicit local escape hatch, not the CI setting.

## Coverage

JaCoCo is wired into the default `test` task:

```bash
./gradlew test jacocoTestReport
```

Reports land at `build/reports/jacoco/test/html/index.html` and `build/reports/jacoco/test/jacocoTestReport.xml`. The Linux leg of the PR-gate matrix uploads the XML report as a CI artifact for later aggregation.

## Building from Source

The build produces two artifacts (see PortSwigger's [extension-portal issue #231](https://github.com/PortSwigger/extension-portal/issues/231)):

```bash
./gradlew clean shadowJar                  # full build (default, GitHub releases)
./gradlew clean shadowJar -PstoreBuild=true # BApp Store build
```

* **Full build** → `build/libs/Custom-AI-Agent-full-<version>.jar` — registers all 59 MCP tools.
* **Store build** → `build/libs/Custom-AI-Agent-<version>.jar` — registers only the 8 extension-native AI MCP tools (BApp Store compliance).

The `-PstoreBuild` flag sets the generated `BuildFlags.STORE_BUILD` constant that gates which tools register. Use the resulting JAR to install the extension in Burp, where it appears as **Custom AI Agent**.

## SBOM

```bash
./gradlew cyclonedxBom
```

Produces the CycloneDX JSON file `build/reports/sbom/bom.json`. The release workflow attaches it to the GitHub Release alongside the JAR and its SHA-256 checksum.

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
3.  Check `audit.jsonl` (if Audit Logging is enabled) to inspect the captured dispatch text and response events. Remember that separate system-role profile text and reconstructed conversation history are not serialized into the prompt bundle.

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
- Redaction policy mapping, RFC 5869 host pseudonyms, structured cookie/header/parameter carriers, serialized MCP results, and scanner issue-detail ownership.
- Injection point extraction for escaped strings, booleans, and nulls.
- Active payload generation paths (numeric/string/UUID).
- Response analyzer diff and time-based detection boundaries.
- Shared HTTP conversation history trimming and concurrent writes.
- MCP executor EDT confinement with assertions both enabled and disabled.
