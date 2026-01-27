# Prompt Defaults

These are the default prompt templates used for the context menu actions. You can customize these in **Settings → Prompt Templates**.

## Request-Based Prompts

### Find Vulnerabilities
> Analyze this HTTP request/response for security vulnerabilities. Check for: injection points (SQLi, XSS, CMDI, SSTI, SSRF), authentication/authorization flaws (IDOR, BOLA, BAC), information disclosure, insecure headers/cookies, sensitive data exposure, misconfigurations. For each finding provide: vulnerability type, evidence, severity (CVSS), exploitation steps, and remediation.

### Quick Recon
> Summarize this endpoint for recon: HTTP method, path, authentication mechanism, all parameters (query/body/headers/cookies), response data type and key fields, notable security observations. Format: 5-7 concise bullets.

### Explain JS
> Summarize JS behavior and any security impact. Output: bullets + 1 risk note.

### Access Control
> Design an access-control test plan for this request: horizontal/vertical escalation, missing authorization checks, auth bypass. For each test, give the modified request and expected outcome.

### Login Sequence
> Draft a login sequence from this traffic. Output: steps + parameters to capture.

## Issue-Based Prompts

### Analyze this Issue
> Analyze the finding. Explain the vulnerability and root cause, cite concrete evidence from the request/response, and list precise validation steps.

### Generate PoC & Validate
> Provide a step-by-step PoC with exact HTTP requests (curl where possible), expected responses, and safe validation criteria.

### Impact & Severity
> Assess impact and severity: CIA impact, exploitability, likely business risk, and CVSS considerations. Keep it concise.

### Full Report
> Write a complete vulnerability report: summary, root cause, evidence, impact, PoC, remediation.