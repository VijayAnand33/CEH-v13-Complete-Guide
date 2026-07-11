# Metasploit

## Overview

Metasploit is a powerful open-source penetration testing framework developed by Rapid7 that provides a modular platform for vulnerability assessment, exploit development, post-exploitation, and security testing. In addition to exploitation, it includes numerous auxiliary modules for network reconnaissance, port scanning, service enumeration, and information gathering, making it a versatile tool throughout the penetration testing lifecycle.

---

## Purpose

Metasploit is used to:

- Perform network reconnaissance and information gathering.
- Identify active hosts, open ports, and running services.
- Validate and exploit known vulnerabilities.
- Execute post-exploitation activities.
- Develop and test exploits and payloads.
- Support penetration testing and security research.

---

## Key Features

- Modular architecture with exploits, payloads, auxiliary modules, encoders, and post-exploitation modules.
- Integrated database for managing scan results and discovered hosts.
- Auxiliary modules for reconnaissance and network scanning.
- Large collection of publicly available exploits.
- Payload generation and customization.
- Automation through resource scripts.
- Integration with external tools such as Nmap.

---

## Installation

### Debian / Ubuntu / Parrot OS

```bash
sudo apt update
sudo apt install metasploit-framework
```

Launch Metasploit:

```bash
msfconsole
```

---

## Basic Syntax

Start the Metasploit console:

```bash
msfconsole
```

Search for a module:

```bash
search <keyword>
```

Load a module:

```bash
use <module_path>
```

Display module options:

```bash
show options
```

Set module parameters:

```bash
set <parameter> <value>
```

Execute the module:

```bash
run
```

Return to the main console:

```bash
back
```

---

## Payload Generation (MSFVenom)

MSFVenom is the payload generation utility included within the Metasploit Framework. It allows penetration testers to generate payloads for multiple platforms and architectures before deploying them during authorized security assessments.

---

## Commonly Used Commands

| Command | Description |
|---------|-------------|
| `msfconsole` | Launch the Metasploit Framework console |
| `search <keyword>` | Search available modules |
| `use <module>` | Load a module |
| `show options` | Display configurable module options |
| `set <option> <value>` | Configure module parameters |
| `run` | Execute the loaded module |
| `back` | Return to the previous menu |
| `info` | Display detailed information about the selected module |
| `help` | Display available commands |

---

## Typical Workflow

```mermaid
flowchart LR
    A[Launch msfconsole] --> B[Search Module]
    B --> C[Load Module]
    C --> D[Configure Options]
    D --> E[Execute Module]
    E --> F[Analyze Results]

    style A fill:#1f2937,color:#fff,stroke:#2563eb
    style B fill:#374151,color:#fff,stroke:#2563eb
    style C fill:#4b5563,color:#fff,stroke:#2563eb
    style D fill:#7c3aed,color:#fff,stroke:#8b5cf6
    style E fill:#059669,color:#fff,stroke:#10b981
    style F fill:#2563eb,color:#fff,stroke:#60a5fa
```

---

## CEH Practical Example

In **Module 03 – Scanning Networks**, Metasploit was used as a reconnaissance platform rather than an exploitation framework. Auxiliary scanning modules were employed to perform SYN port scanning, TCP port scanning, and SMB version discovery against target hosts. These modules helped identify active systems, enumerate open ports, and determine operating system information, complementing the results obtained through Nmap.

In **Module 06 – System Hacking**, Metasploit was extensively used throughout the system hacking lifecycle to establish Meterpreter sessions, generate listeners, bypass User Account Control (UAC), elevate privileges, deploy persistence mechanisms such as Sticky Keys and Run Registry modifications, and maintain remote access to compromised Windows systems.

---

## Advantages

- Comprehensive penetration testing framework.
- Extensive collection of community and commercial modules.
- Supports reconnaissance, exploitation, and post-exploitation.
- Modular and highly extensible architecture.
- Integrates well with other security tools.
- Widely adopted in professional penetration testing.

---

## Limitations

- Requires proper authorization before use on target systems.
- Large framework with a learning curve for beginners.
- Some modules may become outdated as vulnerabilities are patched.
- Aggressive scanning and exploitation activities may trigger IDS/IPS alerts.

---

## Best Practices

- Always obtain proper authorization before scanning or exploiting systems.
- Keep the framework updated to access the latest modules and fixes.
- Validate reconnaissance results before selecting exploits.
- Configure module parameters carefully to avoid unintended network disruption.
- Document findings and preserve scan results for reporting.

---

## Used In

- Module 03 – Scanning Networks
- Module 06 – System Hacking
- Module 10 – Denial-of-Service

---

## References

- https://www.metasploit.com/
- https://docs.rapid7.com/metasploit/
- https://github.com/rapid7/metasploit-framework

---

