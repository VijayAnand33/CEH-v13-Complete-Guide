# WPScan

## Overview

WPScan is a WordPress security scanner designed to find vulnerabilities in WordPress core, plugins, and themes. It leverages a vulnerability database (WPVulnDB) and supports enumeration, fingerprinting, and reporting to assist security assessments of WordPress sites.

---

## Purpose

The primary purpose of WPScan is to automate the discovery of WordPress-specific security issues such as outdated plugins/themes, vulnerable versions, and exposed usernames to help prioritize manual verification and remediation.

---

## Key Features

- WordPress fingerprinting (version detection)
- Plugin and theme enumeration
- Vulnerability lookup via WPVulnDB (API token)
- Username enumeration
- Multiple output formats and report generation
- Support for passive and active detection methods

---

## Installation

### Linux (Debian / Kali)

```bash
sudo apt install wpscan
```

### Ruby gem (alternative)

```bash
sudo gem install wpscan
```

### Verify Installation

```bash
wpscan --version
```

---

## Basic Usage

1. Enumerate plugins and attempt vulnerability checks:

```bash
wpscan --url https://example.com --enumerate p --api-token YOUR_WPVULNDB_TOKEN
```

2. Enumerate users and plugins, save output:

```bash
wpscan --url https://example.com --enumerate u,p --format json --output wpscan-report.json
```

---

## Common Options

| Option | Description |
|--------|-------------|
| `--url` | Target WordPress site URL. |
| `--enumerate` | What to enumerate: `u` (users), `p` (plugins), `t` (themes), `vp` (vulnerable plugins), `tt` (timthumbs), etc. Use commas to combine. |
| `--plugins-detection` | Plugin detection method: `passive` or `mixed`/`aggressive`. |
| `--plugins-version-detection` | Attempt to detect plugin versions for vulnerability correlation. |
| `--api-token` | WPVulnDB API token to enable vulnerability lookups. |
| `--format` | Output format: `cli`, `json`, `xml`. |
| `--output` | Write scan output to a file. |
| `--random-agent` | Use a random User-Agent string for requests. |
| `--threads` | Number of concurrent threads (speeds up scanning). |
| `--no-banner` | Suppress the WPScan banner. |

---

## Typical Workflow

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '#D0E8FF', 'primaryTextColor': '#092B4B', 'primaryBorderColor': '#1C6DD0', 'lineColor': '#2A7EE0', 'secondaryColor': '#EBF5FF', 'tertiaryColor': '#BBDFFF'}}}%%
flowchart TD
    A["Target WordPress Site"] --> B["Run WPScan"]
    B --> C["Enumerate Users, Plugins & Themes"]
    C --> D["Identify Potential Vulnerable Components"]
    D --> E["Lookup Vulnerabilities (WPVulnDB)"]
    E --> F{"Vulnerable Component Found?"}
    F -->|Yes| G["Manually Validate Findings"]
    F -->|No| H["Extend Scan Configuration"]
    H --> C
    G --> I["Generate Report & Remediation Plan"]

    classDef start fill:#D6E9FF,stroke:#145DA0,stroke-width:2px,color:#092B4B;
    classDef process fill:#FFFFFF,stroke:#1C6DD0,stroke-width:1.5px,color:#0D1B2A;
    classDef decision fill:#FFF4CC,stroke:#E09F3E,color:#BF360C;
    classDef endNode fill:#DCEED9,stroke:#2E7D32,stroke-width:2px,color:#1B5E20;

    class A,B start;
    class C,D,E,H process;
    class F decision;
    class I endNode;
```

---

## CEH Practical Example

During Module 14 – Web Application Hacking, WPScan can be used to enumerate plugins and identify a plugin with a known CVE, which is then validated manually before attempting any controlled exploitation.

Example:

```bash
wpscan --url https://target.example --enumerate p --api-token $WPVULNDB_TOKEN --format json --output findings.json
```

---

## Advantages

- WordPress-specific checks and large vulnerability database.
- Simple command-line interface suitable for automation.
- Supports API integration for up-to-date vulnerability data.

---

## Limitations

- May produce false positives; manual verification required.
- Aggressive enumeration can be noisy and detectable.
- Requires WPVulnDB API token for full vulnerability lookup functionality.

---

## Best Practices

- Always obtain authorization before scanning WordPress sites.
- Use `--random-agent` and throttle requests to avoid disrupting services.
- Complement WPScan results with manual validation and additional tools.
- Keep WPScan and its vulnerability data up to date.

---

## Used In

- Module 14 – Web Application Hacking

---

## References

- Official: https://wpscan.com/
 - Documentation: https://github.com/wpscanteam/wpscan