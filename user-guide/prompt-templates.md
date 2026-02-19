# Prompt Templates

Prompt templates define default instructions for context menu actions. They are sent together with selected Burp context.

Edit them in **Prompt Templates** in the bottom settings panel.

## Default Template Style

Built-in templates use a structured format with explicit sections (`ROLE` / `TASK` / `SCOPE` / `OUTPUT`) to improve consistency and reduce speculative output.

Use [Prompt Defaults](../reference/prompt-defaults.md) to review exact built-in defaults.

## Built-In Request Prompts

These templates are used for request/response actions:

| Template | Used By | Purpose |
| :--- | :--- | :--- |
| **Find Vulnerabilities** | `Find vulnerabilities` | Broad security analysis across injection, auth/access, disclosure, and configuration issues. |
| **Analyze this request** | `Analyze this request` | Concise endpoint summary. |
| **Explain JS** | `Explain JS` | JavaScript behavior and risk analysis. |
| **Access Control** | `Access control` | Authorization testing guidance. |
| **Login Sequence** | `Login sequence` | Login flow extraction and replay guidance. |

## Built-In Issue Prompts

These templates are used for scanner issue actions:

| Template | Used By | Purpose |
| :--- | :--- | :--- |
| **Analyze this Issue** | `Analyze this issue` | Root cause analysis and validation steps. |
| **Generate PoC & Validate** | `Generate PoC & validate` | Step-by-step PoC with expected evidence. |
| **Impact & Severity** | `Impact & severity` | Impact and severity reasoning. |
| **Full Report** | `Full report` | Complete report structure for delivery. |

## BountyPrompt Integration Controls

The same tab includes BountyPrompt controls:

* **Enable BountyPrompt actions**: Shows/hides curated submenu actions in request/response context menus.
* **Prompt directory**: Filesystem location containing BountyPrompt JSON prompt files.
* **Auto-create issues**: Enables automatic Burp issue creation for eligible BountyPrompt outputs.
* **Issue confidence threshold**: Minimum confidence score (0-100) required for automatic issue creation.
* **Enabled prompt IDs**: Comma- or newline-separated allowlist of curated IDs.

See [BountyPrompt Actions](bountyprompt-actions.md) for operational behavior and curated IDs.

## Guide: Prompt Engineering for Pentesters

### Role Prompting

Start with a clear role definition, for example:

> Act as a senior offensive security expert specialized in web application penetration testing.

### Evidence-Based Reasoning

Require concrete evidence:

> Always cite specific header values, parameter names, or response patterns to justify findings.

### Output Formatting

Request a stable structure:

> Provide findings in Markdown with sections for Type, Evidence, Severity, Impact, and Remediation.

### Language Control

To keep team output consistent:

> Always answer in English.

### Scope Limiting

Reduce speculation:

> Only report findings supported by evidence in the provided request/response context.

## Customization Workflow

1. Open **Prompt Templates** in the bottom settings panel.
2. Edit the desired template text.
3. Run a context action to validate result quality.
4. Iterate until output quality matches your workflow.

Tip: clearing a template field falls back to built-in defaults.
