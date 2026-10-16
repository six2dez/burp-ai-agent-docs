# Testing & Debugging

## Running Tests

The project uses JUnit Platform with JUnit Jupiter `6.0.3` and a Java 21 toolchain.

### Store and full test suites

The two builds are tested separately, each on its own classpath:

| Task | What it runs |
| :--- | :--- |
| `test` | `src/test` on the BApp Store classpath (`src/main` only). |
| `fullTest` | `src/fullTest` on the full classpath (`src/main` + `src/full`). |
| `storeJarContentTest` | Builds both JARs and fails if the BApp Store JAR contains a full-only class or a CLI backend. |
| `unloadLeakTest` | The load/unload lifecycle tests, one JVM per class. |
| `edtGuardWithoutAssertionsTest` | The MCP executor's EDT guard with JVM assertions disabled. |
| `check` | `test`, `fullTest`, `storeJarContentTest`, `unloadLeakTest`, coverage verification, `ktlintCheck` and `detekt`. |

### Fast suite (default in CI PR gate)
```bash
./gradlew test fullTest -PexcludeHeavyTests=true
```
Excludes `*IntegrationTest`, `*ConcurrencyTest`, `*BackpressureTest`, `*RestartPolicyTest`, and `*SupervisionTest` from both suites. It runs on every PR across Linux, macOS, and Windows in `.github/workflows/build.yml`, followed by `edtGuardWithoutAssertionsTest`, `unloadLeakTest` and `bappJar fullJar storeJarContentTest`.

### Nightly-equivalent local suite
```bash
./gradlew test nightlyRegressionTest nightlyRegressionFullTest edtGuardWithoutAssertionsTest unloadLeakTest fullJar bappJar storeJarContentTest
```
This adds the integration, concurrency, backpressure, restart-policy, and supervision suites for both builds (`nightlyRegressionTest` on the store classpath, `nightlyRegressionFullTest` on the full classpath), proves the MCP executor's EDT guard with JVM assertions disabled, runs the unload lifecycle tests, and builds and inspects both JARs. It is the task list the nightly workflow (`.github/workflows/nightly-regression.yml`) runs. On each version tag the release workflow runs `ktlintCheck` and `detekt`, then `test fullTest nightlyRegressionTest nightlyRegressionFullTest edtGuardWithoutAssertionsTest unloadLeakTest`, then `bappJar fullJar storeJarContentTest`, then `cyclonedxBom`.

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

The repository produces two builds, each with its own Gradle task (see PortSwigger's [extension-portal issue #231](https://github.com/PortSwigger/extension-portal/issues/231)):

```bash
./gradlew clean fullJar   # GitHub full build
./gradlew clean bappJar   # BApp Store build
```

* **GitHub full build** → `build/libs/Custom-AI-Agent-full-<version>.jar` — registers all 59 MCP tools.
* **BApp Store build** → `build/libs/custom-ai-agent-bapp.jar` — registers only the 8 extension-native AI MCP tools.

A build's contents are decided by source set, not by a Gradle property or a generated constant: `src/main` is the BApp Store build, and `src/full` adds the full-only plug-ins on top of it. `bappJar` packages `src/main` alone; `fullJar` packages `src/main` + `src/full`; `./gradlew build` builds both. Use the resulting JAR to install the extension in Burp, where it appears as **Custom AI Agent** in both builds.

Each GitHub release attaches `Custom-AI-Agent-full-<version>.jar` and `Custom-AI-Agent-bapp-<version>.jar` (a byte copy of `custom-ai-agent-bapp.jar`), each with a `.sha256` file, plus `bom.json`.

## SBOM

```bash
./gradlew cyclonedxBom
```

Produces the CycloneDX JSON file `build/reports/sbom/bom.json`. The release workflow attaches it to the GitHub Release alongside both JARs and their SHA-256 checksums. One `bom.json` covers both builds: it lists the full build's runtime dependencies, which include every dependency the BApp Store build packages.

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
