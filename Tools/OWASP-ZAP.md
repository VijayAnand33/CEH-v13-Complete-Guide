# OWASP ZAP

## Overview

OWASP Zed Attack Proxy (ZAP) is an open-source web application security testing tool developed by the Open Worldwide Application Security Project (OWASP). It helps security professionals identify vulnerabilities in web applications through automated scanning, spidering, proxy interception, and manual testing.

---

## Purpose

The primary purpose of OWASP ZAP is to assess the security of web applications by discovering application content, identifying vulnerabilities, intercepting HTTP/HTTPS traffic, and assisting penetration testers throughout the testing process.

---

## Key Features

- Intercepting Proxy
- Automated Spider (Crawler)
- Passive Vulnerability Scanning
- Active Vulnerability Scanning
- HTTP Request and Response Inspection
- Session Management
- Reporting
- Extensible through Add-ons

---

## Installation

### Linux

```bash
sudo apt install zaproxy
```

### Windows

Download the latest installer from the official OWASP ZAP website and complete the installation.

### Verify Installation

```bash
zaproxy
```

or launch it from the applications menu.

---

## Basic Usage

1. Launch OWASP ZAP.
2. Configure the browser to use ZAP as the proxy.
3. Browse the target web application.
4. Spider the application.
5. Perform passive or active scans.
6. Analyze the findings.

---

## Common Features
| Feature | Description |
|---------|-------------|
| **Intercepting Proxy** | Captures and inspects HTTP(S) traffic between the browser and the target application for manual analysis and modification. |
| **Spider (Crawler)** | Automatically crawls the web application to discover pages, links, and parameters to build a site map. |
| **Passive Scan** | Analyzes traffic without sending additional requests to detect low-risk issues and informational findings. |
| **Active Scan** | Actively probes the application with crafted payloads to detect exploitable vulnerabilities. |
| **Site Tree** | Visual representation of discovered URLs and application structure for easy navigation and analysis. |
| **Alerts** | Structured findings with severity, description, evidence, and remediation guidance. |
| **Report Generation** | Exports scan results into various report formats (HTML, XML) for sharing and documentation. |

---

## Typical Workflow
```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '#EAF2FB', 'primaryTextColor': '#0D1B2A', 'primaryBorderColor': '#145DA0', 'lineColor': '#1B98E0', 'secondaryColor': '#F1F6F9', 'tertiaryColor': '#A6D8FF'}}}%%
flowchart TD
	A[Launch OWASP ZAP] --> B[Configure Browser Proxy]
	B --> C[Browse Target Application]
	C --> D[Spider the Website]
	D --> E[Passive Analysis]
	E --> F[Active Vulnerability Scan]
	F --> G[Review Security Findings]

	classDef start fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B5E20;
	classDef process fill:#E3F2FD,stroke:#1565C0,stroke-width:1.5px,color:#0D47A1;
	classDef endNode fill:#F3E5F5,stroke:#6A1B9A,stroke-width:2px,color:#4A148C;

	class A start;
	class B,C,D,E,F process;
	class G endNode;
```

---

## CEH Practical Example

During **Module 14 – Web Application Hacking**, OWASP ZAP was used to perform **web spidering** on the target application. The spider automatically crawled the website by following hyperlinks and accessible resources, creating a structured site map that revealed pages, directories, and application endpoints for further security assessment.

---

## Advantages

- Free and open-source.
- Beginner-friendly interface.
- Supports both automated and manual testing.
- Excellent for discovering hidden pages and application structure.
- Generates detailed vulnerability reports.
- Widely used in professional web application security testing.

---

## Limitations

- Active scanning may generate significant traffic against the target.
- Some advanced features require experience to interpret correctly.
- Automated findings should always be manually verified.
- Performance depends on application size and complexity.

---

## Best Practices

- Always obtain authorization before scanning a web application.
- Configure the proxy correctly before testing.
- Perform spidering before active scanning.
- Validate all reported vulnerabilities manually.
- Keep OWASP ZAP updated with the latest add-ons and signatures.

---

## Used In

- Module 14 – Web Application Hacking

*(This list will be expanded as additional modules are completed.)*

---

## Related Tools

- Burp Suite
- Wapiti
- SmartScanner
- Nmap

---

## References

- Official Website: https://www.zaproxy.org/
- Official Documentation: https://www.zaproxy.org/docs/