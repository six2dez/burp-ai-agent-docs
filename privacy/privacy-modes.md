# Privacy Modes

Privacy modes define what data can leave Burp when the extension calls AI backends.

Configure them in **Privacy & Logging** in the bottom settings panel.

## Modes

### STRICT

Designed for high-sensitivity engagements.

* **Hostnames**: Anonymized with stable pseudonyms.
* **Auth tokens**: Redacted.
* **Cookies**: Stripped/redacted.

Typical use: cloud backends where host identity must not be exposed.

### BALANCED

Balances context quality and basic hygiene.

* **Hostnames**: Preserved.
* **Auth tokens**: Redacted.
* **Cookies**: Stripped/redacted.

Typical use: local or approved environments where hostnames can be shared.

### OFF

No redaction.

* Full raw request/response context is eligible for transmission.

Typical use: controlled internal testing where maximum context is required.

![Screenshot: Privacy settings](../.gitbook/assets/privacy-settings.png)

## Important Notes

* Privacy mode governs prompt context sent to AI backends.
* Active scanning still sends real traffic to real targets.
* Determinism mode preserves stable redaction mapping behavior.
* BountyPrompt actions apply redaction before tag resolution, so each `[HTTP_*]` segment inherits the active privacy policy.
