# Wapiti

## Overview

Wapiti is an open-source web application vulnerability scanner that performs black-box scans by injecting payloads into web application parameters and analyzing responses for potential vulnerabilities. It supports multiple modules for detecting issues such as XSS, SQL injection, file disclosure, and more.

---

## Purpose

Wapiti automates the detection of web application vulnerabilities by crawling target sites, testing input points with crafted payloads, and reporting suspicious responses for manual verification.

---

## Key Features

- Web crawling and parameter discovery
- Detection modules for XSS, SQL injection, file disclosure, command injection, and more
- Report generation in multiple formats (HTML, XML)
- Customizable attack modules and payloads
- Supports authentication and session handling

---

## Installation

### Linux

```bash
sudo apt install wapiti
```

### Verify Installation

```bash
wapiti --version
```

---

## Basic Usage

```bash
wapiti http://example.com -o report.html
```

To enable specific modules:

```bash
wapiti -u http://example.com -m xss,sqli -o findings.html
```

---

## Common Options

| Option | Description |
|--------|-------------|
| `-u` / positional URL | Target URL to scan. |
| `-m` | Comma-separated modules to run (e.g., `xss,sqli`). |
| `-o` | Output file (HTML, XML). |
| `-f` | Output format (html, xml). |
| `--scope` | Limit scanning to a specific host or path. |
| `--auth` | Provide authentication credentials or use session cookies. |
| `--crawl` | Control crawler depth and behavior. |
| `--timeout` | Set request timeout. |

---

## Typical Workflow

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '#D0E8FF', 'primaryTextColor': '#092B4B', 'primaryBorderColor': '#1C6DD0', 'lineColor': '#2A7EE0', 'secondaryColor': '#EBF5FF', 'tertiaryColor': '#BBDFFF'}}}%%
flowchart TD
	A["Select Target Site"] --> B["Run Wapiti Crawler"]
	B --> C["Discover Parameters & Inputs"]
	C --> D["Run Detection Modules (XSS, SQLi, etc.)"]
	D --> E["Collect Suspicious Responses"]
	E --> F{"Findings Suspicious?"}
	F -->|Yes| G["Generate Report"]
	F -->|No| H["Refine Scan or Parameters"]
	H --> C
	G --> I["Manually Validate Findings"]

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

In Module 14 – Web Application Hacking, Wapiti was used to scan the target application for common input-based vulnerabilities and to generate a report for manual follow-up.

---

## Advantages

- Lightweight and easy to use.
- Multiple detection modules for common vulnerabilities.
- Generates readable reports suitable for triage.

---

## Limitations

- May produce false positives that require manual validation.
- Not as feature-rich as some commercial scanners.
- Scanning can be noisy and should be authorized.

---

## Best Practices

- Obtain authorization before scanning.
- Limit the crawler scope to avoid unintended content exposure.
- Combine automated results with manual testing.
- Review and validate all reported findings.

---

## Used In

- Module 14 – Web Application Hacking

---

## References

- Official: https://wapiti-scanner.github.io/
- Official Documentation: https://wapiti-scanner.github.io/documentation.html