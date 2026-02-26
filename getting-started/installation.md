# Installation

## Prerequisites

1. **Burp Suite** Community or Professional (`2023.12+` recommended).
2. **Java 21** for building from source.

> Recent Burp versions include bundled Java runtime for extension execution. Separate Java is mainly needed for local builds.

## Install Path

{% tabs %}
{% tab title="Download from Releases" %}
1. Open [GitHub Releases](https://github.com/six2dez/burp-ai-agent/releases).
2. Download latest `Burp-AI-Agent-<version>.jar`.
{% endtab %}

{% tab title="Build from Source" %}
1. Clone repository:

```bash
git clone https://github.com/six2dez/burp-ai-agent.git
cd burp-ai-agent
```

2. Build fat JAR:

```bash
./gradlew clean shadowJar
```

3. Output path:

```text
build/libs/Burp-AI-Agent-<version>.jar
```
{% endtab %}
{% endtabs %}

## Load into Burp Suite

1. Open **Extensions -> Installed -> Add**.
2. Select extension type `Java`.
3. Choose the JAR file.
4. Complete load wizard.

![Screenshot: Burp extensions add](../.gitbook/assets/burp-extensions-add.png)

## Verify Installation

Expected indicators:

* extension loads without startup errors,
* **AI Agent** tab appears in Burp main navigation.

<figure><img src="../.gitbook/assets/installation-ai-agent-tab.png" alt="Burp with AI Agent tab visible after extension load"><figcaption></figcaption></figure>

## Runtime Directory

On first start, `~/.burp-ai-agent/` is created:

```text
~/.burp-ai-agent/
├── audit.jsonl
├── bundles/
├── contexts/
├── backends/
├── certs/
│   └── mcp-keystore.p12
└── AGENTS/
    ├── default
    ├── pentester.md
    ├── bughunter.md
    └── auditor.md
```

Custom additions:

* profiles: `~/.burp-ai-agent/AGENTS/*.md` ([Agent Profiles](../user-guide/agent-profiles.md))
* backend plugins: `~/.burp-ai-agent/backends/` ([Adding a Backend](../developer/adding-backend.md))

## Troubleshooting

* Extension load failure: inspect Burp Errors/Output tabs and Java version.
* Tab missing: ensure extension is enabled.
* Permission errors: ensure write access to `~/.burp-ai-agent/`.

## Next Steps

Continue with [Quick Start](quick-start.md).
