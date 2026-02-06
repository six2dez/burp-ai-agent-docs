# Prompt Defaults

These are the built-in prompt templates used by context menu actions. You can customize them in **Prompt Templates tab in the bottom settings panel**.

## Request-Based Prompts

### Find Vulnerabilities
```markdown
### ROLE
Analyze the provided HTTP traffic as a Senior Security Researcher.
Response Language: English.

### TASK
Identify security vulnerabilities, architectural flaws, and business logic issues.

### SCOPE
- **Injections**: SQLi, XSS, Command, Template (SSTI), SSRF, XXE, NoSQL.
- **Auth & Access**: IDOR/BOLA, Broken Authentication, JWT issues, CSRF.
- **Exposure**: PII, Secrets, Debug Info, Source Code leaks.
- **Logic**: Mass Assignment, Race Conditions, Price/Quantity manipulation.

### OUTPUT FORMAT
For each finding, provide:
1. **Type**: Vulnerability category.
2. **Evidence**: Quote the specific code, parameter, or header.
3. **Severity**: CVSS-based (Low, Medium, High, Critical).
4. **Impact**: Potential consequences.
5. **Remediation**: Actionable fix.
```

### Analyze this request
```markdown
### TASK
Summarize the security profile of this endpoint.
Response Language: English.

### FORMAT (5-7 bullets)
- **Endpoint**: Method and Path purpose.
- **Authentication**: Mechanism used (JWT, Session, API Key).
- **Inputs**: Notable query, body, or header parameters.
- **Data Flow**: Type of data returned and its sensitivity.
- **Security Observations**: Any immediate red flags or good practices noted.
```

### Explain JS
```markdown
### TASK
Analyze the provided JavaScript code for security relevance.
Response Language: English.

### OUTPUT
- **Behavior**: Concise summary of what the code does.
- **Sinks**: Identify usage of dangerous functions (eval, innerHTML, etc.).
- **Sensitive Data**: Identify hardcoded keys, endpoints, or patterns.
- **Risk Note**: One-sentence summary of the security risk.
```

### Access Control
```markdown
### TASK
Design a systematic access control test plan for this request.
Response Language: English.

### TEST MATRIX
1. **Horizontal Escalation**: Accessing same-role data (e.g., `userId=B` instead of `A`).
2. **Vertical Escalation**: Regular user accessing admin functions.
3. **Authentication Bypass**: Request without session/tokens.
4. **Parameter Pollution**: Testing if roles/permissions can be overwritten.

For each test, provide:
- **Modification**: What to change in the request.
- **Expected Outcome**: What behavior would indicate a vulnerability.
```

### Login Sequence
```markdown
### TASK
Map the authentication flow based on the provided traffic.
Response Language: English.

### OUTPUT
- **Step-by-Step Flow**: Sequence of requests to complete login.
- **Session Keys**: Tokens/Headers to capture for persistence.
- **Failure Indicators**: How to detect session expiration.
```

## Issue-Based Prompts

### Analyze this Issue
```markdown
### TASK
Analyze the provided security finding in depth.
Response Language: English.

### REQUIREMENTS
- Explain the **vulnerability mechanics** clearly.
- Identify the **root cause** in the application logic.
- Cite **concrete evidence** from the data provided.
- Provide a list of **manual validation steps** for a researcher.
```

### Generate PoC & Validate
```markdown
### TASK
Generate a step-by-step Proof of Concept (PoC) for validation.
Response Language: English.

### REQUIREMENTS
1. provide exact **HTTP requests** (curl where possible).
2. document the **expected response** indicating success.
3. define **safe validation criteria** to avoid production impact.
```

### Impact & Severity
```markdown
### TASK
Assess the impact and overall risk of this finding.
Response Language: English.

### CRITERIA
- **CIA Impact**: Confidentiality, Integrity, Availability.
- **Exploitability**: Skill level and preconditions required.
- **Business Risk**: Financial, reputational, or operational.
- **CVSS Vector**: Provide a suggested CVSS v3.1 vector.
```

### Full Report
```markdown
### ROLE
Write a professional security vulnerability report.
Response Language: English.

### STRUCTURE
1. **Summary**: Concise overview of the finding.
2. **Root Cause**: Why does this happen? (e.g., lack of sanitization).
3. **Evidence**: Describe the exact request/response behavior.
4. **Impact**: Describe the business and technical risk.
5. **PoC**: Step-by-step reproduction instructions.
6. **Remediation**: Detailed fix recommendation.
```
