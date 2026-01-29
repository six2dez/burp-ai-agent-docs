# Agent Profiles

Agent Profiles allow you to customize the AI's system instructions based on your engagement type. Profiles are Markdown files stored in `~/.burp-ai-agent/AGENTS/` that inject role-specific guidance into every AI interaction.

## Installation

The extension creates the `~/.burp-ai-agent/AGENTS/` directory automatically, but the profile files themselves are **not** auto-generated. Copy them from the repository:

```bash
cp -r AGENTS/ ~/.burp-ai-agent/AGENTS/
```

This copies three built-in profiles: `pentester.md`, `bughunter.md`, and `auditor.md`. Without these files, the agent profile feature silently does nothing — no error is shown, but no system instructions are injected.

## How It Works

1. You select a profile from the **Agent profile** dropdown in **Settings** (options: `pentester`, `bughunter`, `auditor`).
2. The extension writes the active profile name to `~/.burp-ai-agent/AGENTS/default`.
3. When a chat session or context menu action runs, the extension loads the corresponding `.md` file and prepends its instructions to the AI prompt.

## Profile File Format

Profile files use a simple section-based format with `[SECTION_NAME]` headers:

```markdown
You are an expert penetration tester. Focus on identifying high-impact
vulnerabilities and providing actionable remediation advice.

[REQUEST_ANALYSIS]
When analyzing HTTP requests, prioritize:
- Authentication and authorization flaws
- Injection vulnerabilities (SQLi, XSS, command injection)
- Business logic issues

[ISSUE_ANALYSIS]
When reviewing scanner findings:
- Assess exploitability in the current context
- Provide CVSS scoring rationale
- Suggest concrete remediation steps

[JS_ANALYSIS]
When analyzing JavaScript:
- Look for hardcoded secrets and API keys
- Identify client-side validation that can be bypassed
- Map API endpoints and data flows

[DEFAULT]
Provide detailed technical analysis with evidence.
```

### Structure

* **Global section** (text before any `[SECTION]` header): Injected into every prompt regardless of action.
* **Named sections**: Injected when the corresponding context menu action triggers. The `[DEFAULT]` section is used as a fallback when no specific section matches the action.

## Section-to-Action Mapping

The extension maps context menu actions to profile sections:

| Context Menu Action | Profile Section |
| :--- | :--- |
| Find vulnerabilities | `[REQUEST_ANALYSIS]` |
| Analyze this request | `[ANALYZE_REQUEST]` |
| Explain JS | `[JS_ANALYSIS]` |
| Access control | `[ACCESS_CONTROL]` |
| Login sequence | `[LOGIN_SEQUENCE]` |
| Analyze this issue | `[ISSUE_ANALYSIS]` |
| Generate PoC & validate | `[POC]` |
| Impact & severity | `[ISSUE_IMPACT]` |
| Full report | `[FULL_REPORT]` |
| Free-form chat | `[CHAT]` |

If no matching section is found, the `[DEFAULT]` section is used. If neither exists, only the global section is injected.

## Built-in Profiles

The extension UI offers three profile presets:

| Profile | Description |
| :--- | :--- |
| `pentester` | General-purpose penetration testing focus. Emphasizes exploitation, PoC generation, and remediation. |
| `bughunter` | Bug bounty oriented. Prioritizes impact, severity, and report-ready output. |
| `auditor` | Compliance and audit focus. Emphasizes controls, regulatory frameworks, and documentation. |

## Creating Custom Profiles

1. Navigate to `~/.burp-ai-agent/AGENTS/`.
2. Create a new Markdown file (e.g., `apitester.md`).
3. Write your global instructions and any `[SECTION]` blocks you need.
4. In the extension settings, the dropdown shows the three built-in names. To use a custom profile, either:
   * Name your file to match one of the existing names (e.g., overwrite `pentester.md`).
   * Or manually edit the `~/.burp-ai-agent/AGENTS/default` file to contain your custom filename (e.g., `apitester.md`).

## File Caching

The profile loader caches the parsed profile and checks the file modification timestamp on each use. If you edit a profile file while Burp is running, the changes are picked up automatically on the next AI interaction without needing to restart.

## Tips

* Keep global instructions concise (2-3 sentences) to avoid consuming too much of the model's context window.
* Use section-specific instructions for detailed guidance per action type.
* The `[DEFAULT]` section is a good place for general output formatting preferences.
* Profile instructions appear as "System instructions (AGENTS):" in the prompt sent to the AI.
