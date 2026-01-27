# Sample Prompts

These are example prompts you can use directly in the chat or as inspiration for customizing your prompt templates.

## Vulnerability Analysis

### General Security Review
> "Analyze this HTTP request/response for security vulnerabilities. Focus on: injection points (SQLi, XSS, SSTI), authentication/authorization flaws, information disclosure, and insecure configurations. For each finding, provide the vulnerability type, specific evidence, severity, and remediation."

### SQL Injection Focus
> "Check this request for SQL injection in all parameters. Test for error-based, blind boolean, and time-based detection. Provide specific payloads I can use to verify each finding."

### XSS Analysis
> "Analyze the response for reflected and DOM-based XSS. Identify all user input reflected in the response, check for encoding/sanitization, and suggest bypass payloads for any filters detected."

### SSRF Detection
> "Check if any parameter in this request could be used for SSRF. Look for URL parameters, redirect endpoints, and webhook configurations. Suggest payloads to test for blind SSRF using Collaborator."

## Issue Analysis

### Root Cause Analysis
> "Analyze the finding. Explain the vulnerability and root cause, cite concrete evidence from the request/response, and list precise validation steps a pentester can follow to confirm this is exploitable."

### PoC Generation
> "Provide a step-by-step PoC with exact HTTP requests (curl where possible), expected responses, and safe validation criteria. Include both the vulnerable request and a comparison request showing normal behavior."

### Impact Assessment
> "Assess the impact of this vulnerability: CIA triad analysis, real-world exploitability, likely business impact, and CVSS v3.1 score with vector string. Consider the application context."

## Access Control Testing

### Privilege Escalation Plan
> "Design an access-control test plan for this request: horizontal/vertical escalation, missing authorization checks, auth bypass. For each test, give the modified request and expected outcome. Consider both authenticated and unauthenticated scenarios."

### IDOR Testing
> "Analyze this API endpoint for IDOR vulnerabilities. Identify all object references (IDs, UUIDs, filenames) in the request. For each reference, suggest a test case to verify whether authorization is enforced."

## API Security

### API Endpoint Mapping
> "Summarize this endpoint: HTTP method, path, authentication mechanism, all parameters (query/body/headers/cookies), response data type and key fields, and any notable security observations. Format as a concise table."

### JWT Analysis
> "Analyze the JWT token in this request. Decode the header and payload, check the algorithm, identify any weaknesses (none algorithm, weak signing, sensitive data in payload), and suggest tests."

### GraphQL Security
> "Analyze this GraphQL request for security issues: introspection exposure, authorization bypass via nested queries, injection in variables, batch query abuse, and excessive data exposure."

## Reconnaissance

### Quick Endpoint Summary
> "In 5-7 bullets: what does this endpoint do, what authentication does it use, what parameters does it accept, what data does it return, and what are the security-relevant observations?"

### Technology Fingerprinting
> "Based on the response headers, error messages, and behavior patterns, identify the technology stack: web server, framework, language, database, CDN, WAF, and any version information."

## Report Generation

### Client-Ready Report
> "Write a complete vulnerability report suitable for client delivery. Include: executive summary, technical description, root cause, evidence (with sanitized request/response excerpts), impact analysis, CVSS score, step-by-step remediation, and references."

### Bug Bounty Write-up
> "Write a bug bounty report for this finding. Include: title, severity, description, steps to reproduce (numbered), impact, and suggested fix. Keep it concise and follow common bug bounty platform formatting."
