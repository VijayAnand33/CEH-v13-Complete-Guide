# nslookup

## Overview

nslookup (Name Server Lookup) is a command-line network administration utility used to query the Domain Name System (DNS). It retrieves DNS records such as A, AAAA, CNAME, MX, NS, and PTR records, helping security professionals gather domain and DNS infrastructure information during the Footprinting and Reconnaissance phase.

---

## Purpose

nslookup is primarily used to:

- Perform DNS lookups
- Resolve domain names to IP addresses
- Identify authoritative name servers
- Query various DNS record types
- Troubleshoot DNS issues
- Support passive reconnaissance and DNS enumeration

---

## Key Features

- Interactive and non-interactive modes
- Supports multiple DNS record types
- Domain-to-IP resolution
- Name server identification
- Reverse DNS lookup
- Cross-platform support

---

## Installation

nslookup is included by default in most operating systems.

**Windows**

Included with Windows.

**Linux**

Available through the `dnsutils` package if not already installed.

```bash
sudo apt install dnsutils
```

---

## Basic Syntax

```bash
nslookup

nslookup <domain>

nslookup <domain> <dns-server>
```

---

## Commonly Used Commands

| Command | Description |
|---------|-------------|
| `nslookup` | Starts interactive mode. |
| `set type=a` | Queries IPv4 (A) records. |
| `set type=aaaa` | Queries IPv6 (AAAA) records. |
| `set type=cname` | Queries Canonical Name (CNAME) records. |
| `set type=mx` | Queries Mail Exchange (MX) records. |
| `set type=ns` | Queries Name Server (NS) records. |
| `set type=ptr` | Performs reverse DNS lookup. |
| `exit` | Exits interactive mode. |

---

## Typical Workflow

```mermaid
flowchart LR

A[Launch nslookup]
--> B[Specify DNS Record Type]

B --> C[Query Target Domain]

C --> D[Retrieve DNS Records]

D --> E[Analyze Results]

E --> F[Support Further Reconnaissance]

style A fill:#0d1117,color:#fff,stroke:#58a6ff
style B fill:#161b22,color:#fff,stroke:#58a6ff
style C fill:#161b22,color:#fff,stroke:#58a6ff
style D fill:#161b22,color:#fff,stroke:#58a6ff
style E fill:#161b22,color:#fff,stroke:#58a6ff
style F fill:#238636,color:#fff,stroke:#2ea043
```

---

## CEH Practical Example

In **Module 02 – Footprinting and Reconnaissance**, **nslookup** was used to perform DNS enumeration on the target domain. Queries were executed to retrieve **A records**, **CNAME records**, and identify the **authoritative name server**. The results were used to determine the target's IP address and DNS infrastructure, providing valuable intelligence for subsequent reconnaissance activities.

---

## Advantages

- Built into most operating systems
- Simple and easy to use
- Supports multiple DNS record types
- Useful for DNS troubleshooting
- Valuable during reconnaissance
- Lightweight command-line utility

---

## Limitations

- Limited to publicly available DNS records
- Does not perform advanced DNS enumeration
- Some DNS servers restrict query responses
- Cannot identify internal DNS infrastructure

---

## Best Practices

- Query multiple DNS record types for comprehensive enumeration.
- Verify results using additional DNS reconnaissance tools.
- Compare authoritative and non-authoritative responses.
- Use only against authorized targets.
- Combine nslookup results with WHOIS and DNSDumpster for more complete reconnaissance.

---

## Used In

- Module 02 – Footprinting and Reconnaissance

---

## References

- https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/nslookup
- https://manpages.ubuntu.com/manpages/latest/en/man1/nslookup.1.html
- https://www.ibm.com/docs/en/aix/7.3?topic=n-nslookup-command