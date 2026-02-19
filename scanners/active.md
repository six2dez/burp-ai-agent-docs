# Active AI Scanner

The Active AI Scanner performs **dynamic, targeted tests** against the application. Unlike the Passive Scanner, which only observes, the Active Scanner sends new HTTP requests to probe for vulnerabilities.

> **WARNING: DANGEROUS OPERATIONS**
>
> The Active Scanner sends traffic that can modify data, trigger actions, or disrupt services.
>
> * **Do not use** on production systems unless authorized.
> * **Do not use** "DANGEROUS" risk level without explicit permission.
> * Always ensure the target is within your **Scope**.

## How It Works

1. The scanner receives a target request (from the passive scanner queue, context menu, or manual selection).
2. **Injection points** are automatically extracted from the request (URL params, body params, headers, cookies, JSON fields, XML elements, path segments).
3. For each injection point, the AI selects appropriate payloads based on the vulnerability class, risk level, and scan mode.
4. Payloads are sent and responses are analyzed using multiple detection methods.
5. Confirmed findings are reported as Burp issues with an `[AI Active]` prefix, normalized by vulnerability class (e.g., `[AI Active] SQLI`).

## Targeted Tests (Context Menu)

From the request context menu, **Targeted tests** lets you run focused active checks (SQLi, XSS, SSRF, IDOR, etc.) instead of scanning all classes. This uses the same active scanner pipeline and includes the same safety warnings.

## Risk Levels

The scanner operates in three risk modes. You must select the appropriate level for your engagement.

| Level         | Description                                                     | Examples                                                                    |
| ------------- | --------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **SAFE**      | Read-only payloads. Unlikely to modify state.                   | `sleep(5)` (SQLi time-based), `{{7*7}}` (SSTI), probing for hidden headers. |
| **MODERATE**  | May read sensitive data or bypass auth.                         | `UNION SELECT` queries, accessing `/etc/passwd`, auth bypass attempts.      |
| **DANGEROUS** | **Destructive**. May delete data, drop tables, or create users. | `DROP TABLE`, `rm -rf`, `INSERT INTO users`.                                |

![Screenshot: Active scanner settings](../.gitbook/assets/active-scanner.png)

## Scan Modes

The scan mode determines which vulnerability classes are tested. Choose based on your engagement type.

| Mode            | Description                                                                                              | Use Case                                          |
| --------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| **BUG\_BOUNTY** | Curated subset of high-impact vulnerability classes. Minimizes noise and focuses on reportable findings. | Bug bounty programs, time-limited assessments.    |
| **PENTEST**     | Broader testing set, including information disclosure and lower-severity checks.                        | Penetration tests, compliance audits.             |
| **FULL**        | All 62 vulnerability classes are tested.                                                                 | Broad security assessments, internal applications. |

## Configuration

Configure these options in the **AI Active Scanner** tab of the bottom settings panel.

* **Max Concurrent Scans**: Number of parallel scans (range: 1–10, default: 3). Keep low to avoid WAF blocking or DoS.
* **Max Payloads per Point**: Maximum payload variations per injection point (range: 1–50, default: 10).
* **Timeout**: Seconds to wait for each scan request (range: 5–120, default: 30).
* **Request Delay**: Milliseconds between requests (range: 0–5000, default: 100). Increase to avoid rate limiting.
* **Max Risk Level**: Maximum allowed risk level for payloads (`SAFE`, `MODERATE`, `DANGEROUS`).
* **Scope Only**: **CRITICAL**. Ensure this is checked to prevent scanning out-of-scope assets (e.g., Google Analytics, CDNs).
* **Scan Mode**: Select `BUG_BOUNTY`, `PENTEST`, or `FULL`.
* **Use Collaborator (OAST)**: Enable out-of-band checks. The scanner generates Burp Collaborator payloads and polls for DNS/HTTP interactions.

## Issue Creation Behavior

* Issue details are sanitized to plain text (Markdown formatting removed).
* Issues are named by class (e.g., `[AI Active] SQLI`) and consolidated if an existing issue with the same name and base URL already exists.

## Vulnerability Classes

The Active Scanner tests for **62 vulnerability classes** organized by category.

### Injection (17 classes)

| Class                    | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| `SQLI`                   | SQL Injection (error-based, blind boolean, blind time-based) |
| `XSS_REFLECTED`          | Reflected Cross-Site Scripting                               |
| `XSS_STORED`             | Stored Cross-Site Scripting                                  |
| `XSS_DOM`                | DOM-based Cross-Site Scripting                               |
| `CMDI`                   | OS Command Injection                                         |
| `SSTI`                   | Server-Side Template Injection                               |
| `XXE`                    | XML External Entity Injection                                |
| `LDAP_INJECTION`         | LDAP Injection                                               |
| `XPATH_INJECTION`        | XPath Injection                                              |
| `NOSQL_INJECTION`        | NoSQL Injection (MongoDB, etc.)                              |
| `GRAPHQL_INJECTION`      | GraphQL Injection                                            |
| `LOG_INJECTION`          | Log Injection / Log Forging                                  |
| `LFI`                    | Local File Inclusion                                         |
| `RFI`                    | Remote File Inclusion                                        |
| `PATH_TRAVERSAL`         | Path Traversal / Directory Traversal                         |
| `HOST_HEADER_INJECTION`  | Host Header Injection                                        |
| `EMAIL_HEADER_INJECTION` | Email Header Injection                                       |

### Access Control (9 classes)

| Class                   | Description                          |
| ----------------------- | ------------------------------------ |
| `IDOR`                  | Insecure Direct Object Reference     |
| `BOLA`                  | Broken Object Level Authorization    |
| `BFLA`                  | Broken Function Level Authorization  |
| `BAC_HORIZONTAL`        | Horizontal Broken Access Control     |
| `BAC_VERTICAL`          | Vertical Broken Access Control       |
| `MASS_ASSIGNMENT`       | Mass Assignment / Auto-binding       |
| `SSRF`                  | Server-Side Request Forgery          |
| `CORS_MISCONFIGURATION` | CORS Misconfiguration (passive-only) |
| `DIRECTORY_LISTING`     | Directory Listing Enabled            |

### Authentication Failures (7 classes)

| Class                    | Description                                           |
| ------------------------ | ----------------------------------------------------- |
| `JWT_WEAKNESS`           | JWT Algorithm Confusion, None Algorithm, Weak Signing |
| `AUTH_BYPASS`            | Authentication Bypass                                 |
| `SESSION_FIXATION`       | Session Fixation                                      |
| `WEAK_SESSION_TOKEN`     | Weak or Predictable Session Tokens                    |
| `ACCOUNT_TAKEOVER`       | Account Takeover vectors                              |
| `OAUTH_MISCONFIGURATION` | OAuth/OIDC Misconfiguration                           |
| `MFA_BYPASS`             | Multi-Factor Authentication Bypass                    |

### Security Misconfiguration (5 classes)

| Class                      | Description                                        |
| -------------------------- | -------------------------------------------------- |
| `DEBUG_ENDPOINT`           | Exposed Debug Endpoints                            |
| `STACK_TRACE_EXPOSURE`     | Stack Trace / Error Disclosure                     |
| `VERSION_DISCLOSURE`       | Server/Framework Version Disclosure (passive-only) |
| `MISSING_SECURITY_HEADERS` | Missing Security Headers (passive-only)            |
| `VERBOSE_ERROR`            | Verbose Error Messages                             |

### Integrity Failures (4 classes)

| Class                      | Description                               |
| -------------------------- | ----------------------------------------- |
| `DESERIALIZATION`          | Insecure Deserialization (passive-only)   |
| `REQUEST_SMUGGLING`        | HTTP Request Smuggling (passive-only)     |
| `CSRF`                     | Cross-Site Request Forgery (passive-only) |
| `UNRESTRICTED_FILE_UPLOAD` | Unrestricted File Upload (passive-only)   |

### Insecure Design (4 classes)

| Class                   | Description                   |
| ----------------------- | ----------------------------- |
| `BUSINESS_LOGIC`        | Business Logic Flaws          |
| `RATE_LIMIT_BYPASS`     | Rate Limiting Bypass          |
| `PRICE_MANIPULATION`    | Price / Quantity Manipulation |
| `RACE_CONDITION_TOCTOU` | Race Condition (TOCTOU)       |

### Cryptographic Failures (3 classes)

| Class                | Description                          |
| -------------------- | ------------------------------------ |
| `INSECURE_COOKIE`    | Insecure Cookie Flags (passive-only) |
| `SENSITIVE_DATA_URL` | Sensitive Data in URL                |
| `WEAK_CRYPTO`        | Weak Cryptographic Algorithms        |

### Cache Attacks (2 classes)

| Class             | Description         |
| ----------------- | ------------------- |
| `CACHE_POISONING` | Web Cache Poisoning |
| `CACHE_DECEPTION` | Web Cache Deception |

### Information Disclosure (4 classes)

| Class                  | Description                               |
| ---------------------- | ----------------------------------------- |
| `SOURCEMAP_DISCLOSURE` | Source Map File Exposure (passive-only)   |
| `GIT_EXPOSURE`         | Git Repository Exposure (passive-only)    |
| `BACKUP_DISCLOSURE`    | Backup File Disclosure (passive-only)     |
| `DEBUG_EXPOSURE`       | Debug Information Exposure (passive-only) |

### Cloud / Infrastructure (2 classes)

| Class                 | Description                               |
| --------------------- | ----------------------------------------- |
| `S3_MISCONFIGURATION` | S3 Bucket Misconfiguration (passive-only) |
| `SUBDOMAIN_TAKEOVER`  | Subdomain Takeover (passive-only)         |

### API Security (1 class)

| Class                | Description                          |
| -------------------- | ------------------------------------ |
| `API_VERSION_BYPASS` | Deprecated/legacy API version access |

### Other (4 classes)

| Class              | Description      |
| ------------------ | ---------------- |
| `OPEN_REDIRECT`    | Open Redirect    |
| `HEADER_INJECTION` | Header Injection |
| `CRLF_INJECTION`   | CRLF Injection   |
| `RACE_CONDITION`   | Race Condition   |

> **Note**: Classes marked **(passive-only)** are detected through response analysis and cannot be actively tested with payloads.

## Injection Points

The scanner automatically identifies the following injection point types in each request:

| Type           | Description                         | Example                               |
| -------------- | ----------------------------------- | ------------------------------------- |
| `URL_PARAM`    | Query string parameters             | `?id=123`                             |
| `BODY_PARAM`   | Form body parameters                | `username=admin`                      |
| `HEADER`       | HTTP headers (from allowlist)       | `Host`, `Referer`, `X-Forwarded-Host` |
| `PATH_SEGMENT` | Numeric/UUID/ObjectId path segments | `/api/users/42`                       |
| `COOKIE`       | Cookie values                       | `session=abc123`                      |
| `JSON_FIELD`   | JSON request body fields            | `{"user_id": 1}`                      |
| `XML_ELEMENT`  | XML request body elements           | `<id>1</id>`                          |

## Detection Methods

| Method             | Description                                                             |
| ------------------ | ----------------------------------------------------------------------- |
| **ERROR\_BASED**   | Look for database/framework error messages in responses.                |
| **BLIND\_BOOLEAN** | Compare response differences when injecting true/false conditions.      |
| **BLIND\_TIME**    | Measure response time delays (e.g., `sleep(5)` causing 5s delay).       |
| **REFLECTION**     | Check if the payload is reflected in the response body.                 |
| **OUT\_OF\_BAND**  | DNS/HTTP callback detection via Burp Collaborator.                      |
| **CONTENT\_BASED** | Check for specific content patterns indicating successful exploitation. |

## Deduplication

The scanner prevents duplicate scanning of the same target. If a URL has been scanned within the last **1 hour**, it is skipped automatically. This applies to both manual and auto-queued scans.

## Auto-Queue from Passive

When **Auto-Queue to Active** is enabled in the passive scanner settings, high-confidence passive findings are automatically forwarded to the active scanner queue. This creates a pipeline:

1. Passive scanner identifies a potential vulnerability.
2. Finding is automatically queued for active verification.
3. Active scanner sends targeted payloads to confirm the finding.
4. Confirmed findings are promoted to Burp issues.

## Burp Pro Integration

On Burp Suite Professional, the Active AI Scanner integrates with the native scan engine via `ScanCheck`. This means AI-generated scan checks run alongside Burp's built-in scanner. On Burp Community Edition, the scanner operates independently using manual queue management.

## Recommended Practices

1. **Start Passive**: Let the Passive Scanner find interesting endpoints first.
2. **Verify First**: Use `SAFE` mode to confirm potential injection points without risk.
3. **Escalate Carefully**: Move to `MODERATE` only for confirmed injection points on authorized targets.
4. **Human in the Loop**: Use the `[AI Active]` findings as leads. Always verify them manually using Repeater before reporting.
5. **Monitor Rate Limits**: If the target has WAF or rate limiting, increase the **Request Delay** and reduce **Max Concurrent Scans**.
6. **Use Scope**: Always enable **Scope Only** to prevent accidental scanning of third-party assets.


## Queue Safety and Backpressure

The active scanner enforces queue backpressure to keep runtime stable under heavy workloads:

- A hard queue limit is applied (`ACTIVE_SCAN_MAX_QUEUE_SIZE`, default `2000`).
- New targets are dropped when the queue is full and a diagnostic log is emitted.
- Context-menu initiated active scans now surface queue status more explicitly in the UI.

This complements the existing 1-hour dedup window and scope filtering.
