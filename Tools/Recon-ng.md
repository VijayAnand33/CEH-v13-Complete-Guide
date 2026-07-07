# Recon-ng

## Overview

Recon-ng is an open-source, Python-based web reconnaissance framework designed to automate Open-Source Intelligence (OSINT) gathering. It provides a modular environment with built-in database support, workspace management, reporting capabilities, and numerous reconnaissance modules, enabling security professionals to efficiently collect information about target organizations.

---

## Purpose

Recon-ng is primarily used to:

- Perform automated OSINT reconnaissance
- Discover domains, hosts, and subdomains
- Gather contact information
- Perform reverse DNS lookups
- Manage reconnaissance data using workspaces
- Generate reconnaissance reports
- Support passive information gathering

---

## Key Features

- Modular reconnaissance framework
- Workspace management
- Built-in SQLite database
- Marketplace for installing modules
- Automated data collection
- HTML report generation
- Extensive OSINT modules

---

## Installation

Recon-ng is available on most penetration testing distributions such as Kali Linux and Parrot Security.

To install manually:

```bash
git clone https://github.com/lanmaster53/recon-ng.git

cd recon-ng

pip install -r REQUIREMENTS
```

---

## Basic Syntax

```bash
recon-ng

help

marketplace install all

modules search

modules load <module>

run

back

exit
```

---

## Commonly Used Commands

| Command | Description |
|---------|-------------|
| `help` | Displays available Recon-ng commands. |
| `marketplace install all` | Installs all available modules from the marketplace. |
| `modules search` | Searches available modules. |
| `modules load <module>` | Loads the specified module. |
| `workspaces create <name>` | Creates a new workspace. |
| `workspaces list` | Lists available workspaces. |
| `db insert domains` | Adds a target domain to the workspace database. |
| `show domains` | Displays stored domains. |
| `show hosts` | Displays discovered hosts. |
| `options set <option> <value>` | Configures module options. |
| `run` | Executes the loaded module. |
| `back` | Returns to the previous menu. |
| `exit` | Exits Recon-ng. |

---

## Typical Workflow

```mermaid
flowchart LR

A[Create Workspace]
--> B[Add Target Domain]

B --> C[Load Reconnaissance Module]

C --> D[Configure Module Options]

D --> E[Run Module]

E --> F[Collect & Store Results]

F --> G[Generate HTML Report]

style A fill:#0d1117,color:#fff,stroke:#58a6ff
style B fill:#161b22,color:#fff,stroke:#58a6ff
style C fill:#161b22,color:#fff,stroke:#58a6ff
style D fill:#161b22,color:#fff,stroke:#58a6ff
style E fill:#161b22,color:#fff,stroke:#58a6ff
style F fill:#161b22,color:#fff,stroke:#58a6ff
style G fill:#238636,color:#fff,stroke:#2ea043
```

---

## CEH Practical Example

In **Module 02 – Footprinting and Reconnaissance**, Recon-ng was used to perform OSINT against the target domain by creating a dedicated workspace, adding the target domain, discovering hosts using multiple reconnaissance modules, performing reverse hostname resolution, gathering contact information, identifying subdomains, and generating an HTML reconnaissance report containing the collected results.

---

## Advantages

- Modular and extensible framework
- Centralized workspace management
- Built-in database for storing reconnaissance data
- Automated OSINT collection
- Large collection of reconnaissance modules
- Supports report generation
- Open-source and actively maintained

---

## Limitations

- Primarily focused on passive reconnaissance
- Some modules depend on third-party services or API keys
- Results depend on publicly available information
- Certain modules may become outdated over time

---

## Best Practices

- Create a separate workspace for each engagement.
- Install and update modules before beginning reconnaissance.
- Configure module options carefully before execution.
- Validate collected information using multiple OSINT sources.
- Generate reports to preserve reconnaissance findings.
- Use Recon-ng only against authorized targets.

---

## Used In

- Module 02 – Footprinting and Reconnaissance

---

## References

- https://github.com/lanmaster53/recon-ng
- https://github.com/lanmaster53/recon-ng/wiki