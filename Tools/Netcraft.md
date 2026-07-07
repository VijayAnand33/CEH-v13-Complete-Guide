# Netcraft

## Overview

Netcraft is a web-based reconnaissance service that provides detailed information about websites, domains, hosting infrastructure, SSL certificates, technologies, operating systems, hosting history, and subdomains. It is widely used during the Footprinting and Reconnaissance phase to gather publicly available information about target organizations without directly interacting with their systems.

---

## Purpose

Netcraft is primarily used to:

- Identify website infrastructure
- Discover domains and subdomains
- Gather hosting information
- Identify web server technologies
- View SSL certificate information
- Analyze hosting history and network details
- Support passive reconnaissance activities

---

## Key Features

- Website technology fingerprinting
- Domain and subdomain discovery
- Hosting provider identification
- SSL certificate information
- Hosting history analysis
- Network and infrastructure information
- Passive OSINT reconnaissance

---

## Access

Netcraft is a web-based service and does not require installation.

**Website**

https://www.netcraft.com

---

## Basic Usage

```
Resources
    ↓
Research Tools
    ↓
Site Report
    ↓
Enter Target Domain
    ↓
Analyze Results
```

---

## Commonly Used Features

| Feature | Description |
|----------|-------------|
| Site Report | Displays detailed information about a website |
| Domain Information | Shows domain ownership and infrastructure |
| Subdomain Discovery | Lists publicly identified subdomains |
| Hosting History | Displays historical hosting information |
| Network Details | Shows IP addresses and hosting provider |
| SSL Information | Displays certificate-related details |

---

## Typical Workflow

```mermaid
flowchart LR

A[Enter Target Domain]
--> B[Generate Site Report]

B --> C[Analyze Infrastructure]

C --> D[Identify Subdomains]

D --> E[Collect Network Information]

E --> F[Use Results for Further Reconnaissance]

style A fill:#0d1117,color:#fff,stroke:#58a6ff
style B fill:#161b22,color:#fff,stroke:#58a6ff
style C fill:#161b22,color:#fff,stroke:#58a6ff
style D fill:#161b22,color:#fff,stroke:#58a6ff
style E fill:#161b22,color:#fff,stroke:#58a6ff
style F fill:#238636,color:#fff,stroke:#2ea043
```

---

## CEH Practical Example

In **Module 02 – Footprinting and Reconnaissance**, Netcraft was used to generate a Site Report for the target domain. The report revealed information about the target's infrastructure, hosting environment, technologies used, network details, and publicly available subdomains. This information provided valuable intelligence for subsequent reconnaissance activities.

---

## Advantages

- Passive reconnaissance technique
- No installation required
- Easy to use
- Rich website infrastructure information
- Provides historical hosting data
- Useful for OSINT investigations

---

## Limitations

- Limited to publicly available information
- Some data may not be completely up to date
- Cannot identify non-public assets
- Advanced features may require registration

---

## Best Practices

- Verify information using additional OSINT tools.
- Combine results with DNS enumeration.
- Cross-reference subdomains using multiple sources.
- Use only against authorized targets.
- Treat collected information as reconnaissance data rather than definitive evidence.

---

## Used In

- Module 02 – Footprinting and Reconnaissance

---

## References

- https://www.netcraft.com
- https://www.netcraft.com/platform/site-report/