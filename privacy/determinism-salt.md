# Determinism & Salt

Determinism mode and host anonymization salt are the two controls that make AI workflows reproducible without exposing real hostnames in `STRICT` mode.

## Determinism Mode

When **Determinism Mode** is enabled:

* context bundles are ordered consistently,
* repeated runs over the same selection produce stable prompt structure,
* audit hash comparison across runs becomes meaningful.

This is useful for:

* compliance audits,
* evidence review,
* regression checks on prompt behavior.

## What Determinism Guarantees

Determinism guarantees stable ordering and stable anonymization mappings for equal inputs and equal settings.

It does **not** guarantee:

* identical model answers (provider-side randomness can still exist),
* identical scanner timing,
* identical external network behavior.

## Host Anonymization Salt

The **Host Anonymization Salt** is used in `STRICT` mode to produce deterministic pseudonyms through RFC 5869 HKDF (HMAC-SHA256 extract and expand). The hostname is the input keying material, the configured salt is the HKDF salt, and the application context is `burp-ai-agent:host`.

* Same hostname + same salt -> same pseudonym.
* Same hostname + different salt -> different pseudonym.

Example:

```text
Project A salt: red-team-2026-a
api.customer.tld -> host-c94eca5715a9.local

Project B salt: red-team-2026-b
api.customer.tld -> host-b850547ba166.local
```

The values above follow the current implementation. The format is `host-` plus the first 6 HKDF output bytes encoded as 12 lowercase hexadecimal characters, followed by `.local`.

## Salt Rotation Guidance

Rotate salt when:

* starting a new engagement,
* changing client or environment boundary,
* sharing artifacts with a different audience.

Keep salt unchanged when:

* reproducing findings inside the same engagement,
* comparing deterministic bundles across test runs.

## Interaction with Audit Logging

With **Audit Logging + Determinism** enabled:

* prompt bundles are easier to compare,
* payload hash differences usually indicate real input/policy changes,
* host pseudonyms remain stable for that salt lifecycle.

This combination is recommended for regulated workflows.

## Recommended Defaults

* Cloud backend + sensitive target: `STRICT` + Determinism ON + unique per-engagement salt.
* Local backend + low sensitivity: `BALANCED` + Determinism optional.
* Internal lab only: `OFF` only if raw context sharing is acceptable.

## Common Mistakes

* Reusing the same salt across unrelated client engagements.
* Assuming determinism means model output is always identical.
* Forgetting that active scanner requests still hit real targets.
* Assuming every hostname-shaped field is covered. `STRICT` anonymizes the carriers wired to the host redactor; see [Redaction Coverage and Known Boundaries](limitations.md#redaction-coverage-and-known-boundaries).

## Related Pages

* [Privacy Modes](privacy-modes.md)
* [Redaction Pipeline](../developer/redaction-pipeline.md)
* [Audit Logging](audit-logging.md)
