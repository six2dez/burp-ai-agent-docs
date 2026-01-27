# Limitations & Hallucinations

AI is a useful tool, but it is not a replacement for human judgment. Users must be aware of its limitations.

## Trust but Verify

**Never report a finding from the AI without manual verification.**

### 1. False Positives
AI can "imagine" vulnerabilities based on patterns that appear insecure but are protected by controls the AI cannot see (e.g., a WAF, custom backend filters, or environmental configurations).

**Mitigation**: Always validate findings manually in Repeater. Use the active scanner's `SAFE` mode to send benign probing payloads before escalating.

### 2. False Negatives
The AI may miss vulnerabilities that are present. It can only analyze the data you provide and may not consider:
*   Multi-step attack chains.
*   Race conditions or timing-dependent issues.
*   Business logic flaws that require deep domain knowledge.
*   Vulnerabilities in binary protocols or non-HTTP traffic.

**Mitigation**: Don't rely solely on AI scanning. Use it as one tool alongside manual testing and Burp's native scanner.

### 3. Knowledge Cutoff
AI models have a knowledge cutoff date. They may not be aware of:
*   1-day vulnerabilities or recently published CVEs.
*   New attack techniques published after the training data cutoff.
*   Framework-specific quirks in the latest versions.

**Mitigation**: Supplement AI analysis with up-to-date vulnerability databases (NVD, Snyk, GitHub Advisory).

### 4. Contextual Blindness
The AI only sees the data you provide. It lacks the "big picture" of the application unless it uses MCP tools to explore. Even then, it may miss:
*   Subtle business logic flaws that require deep understanding.
*   Security controls implemented at infrastructure layers (WAF rules, network segmentation).
*   Application state dependencies (e.g., a vulnerability only exploitable after a specific user action).

**Mitigation**: Provide context in your prompts. Use MCP tools to let the AI explore the application structure. Combine AI analysis with human expertise.

### 5. Payload Safety
While the extension enforces risk levels (`SAFE`, `MODERATE`, `DANGEROUS`), the AI may occasionally suggest payloads that:
*   Are more impactful than expected.
*   Trigger unintended side effects.
*   Bypass the risk level classification.

**Mitigation**: Always review AI-generated payloads before sending them to a target, especially in `MODERATE` and `DANGEROUS` modes. Use the active scanner's scope filtering to prevent accidental out-of-scope testing.

### 6. Hallucinated Evidence
AI may generate plausible-sounding but fabricated evidence, such as:
*   Citing response patterns that don't exist in the actual data.
*   Inventing parameter names or header values.
*   Describing behavior that wasn't observed.

**Mitigation**: Cross-reference AI findings with the actual request/response data in Burp. Enable audit logging to preserve the exact data sent to the AI for later verification.

## Model-Specific Considerations

| Model Type | Strengths | Weaknesses |
| :--- | :--- | :--- |
| **Large cloud models** (GPT-4o, Claude, Gemini Pro) | Better reasoning, broader knowledge. | Cost, privacy concerns, API rate limits. |
| **Small local models** (Llama 8B, Mistral 7B) | Free, private, fast. | Limited reasoning depth, may miss subtle vulnerabilities, shorter context windows. |
| **Code-focused models** (CodeLlama, DeepSeek Coder) | Good at JS/code analysis. | May struggle with non-code security concepts. |

## Responsible Use

*   Always have **explicit authorization** before testing any target.
*   Use **privacy modes** to protect sensitive data when using cloud backends.
*   Do not blindly trust AI-generated severity ratings — apply your own CVSS scoring.
*   Document AI-assisted findings differently from manually verified findings in your reports.
*   Use **audit logging** to maintain a traceable record of all AI interactions.
