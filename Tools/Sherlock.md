# Sherlock

## Overview

Sherlock is an open-source Python-based OSINT tool used to discover usernames across numerous social networking platforms and online services. By searching for a specified username, Sherlock identifies associated profiles and provides direct links, enabling security professionals to gather publicly available information during the Footprinting and Reconnaissance phase.

---

## Purpose

Sherlock is primarily used to:

- Discover social media accounts associated with a username
- Perform Open-Source Intelligence (OSINT)
- Identify publicly available online profiles
- Gather information about individuals or organizations
- Support passive reconnaissance investigations

---

## Key Features

- Username enumeration across hundreds of websites
- Direct profile URL discovery
- Fast multi-platform scanning
- Command-line interface
- Open-source and actively maintained
- Cross-platform support

---

## Installation

Sherlock can be installed using Git and Python.

```bash
git clone https://github.com/sherlock-project/sherlock.git

cd sherlock

pip install -r requirements.txt
```

---

## Basic Syntax

```bash
python3 sherlock.py <username>

python3 sherlock.py <username> --output results.txt
```

---

## Commonly Used Options

| Option | Description |
|---------|-------------|
| `<username>` | Username to search across supported websites. |
| `--output <file>` | Saves the results to a file. |
| `--timeout <seconds>` | Sets the timeout for website requests. |
| `--print-found` | Displays only successfully discovered profiles. |
| `--verbose` | Displays detailed execution information. |
| `--help` | Displays available command options. |

---

## Typical Workflow

```mermaid
flowchart LR

A[Enter Username]
--> B[Search Supported Websites]

B --> C[Identify Matching Profiles]

C --> D[Collect Profile URLs]

D --> E[Analyze Public Information]

E --> F[Use Results for OSINT]

style A fill:#0d1117,color:#fff,stroke:#58a6ff
style B fill:#161b22,color:#fff,stroke:#58a6ff
style C fill:#161b22,color:#fff,stroke:#58a6ff
style D fill:#161b22,color:#fff,stroke:#58a6ff
style E fill:#161b22,color:#fff,stroke:#58a6ff
style F fill:#238636,color:#fff,stroke:#2ea043
```

---

## CEH Practical Example

In **Module 02 – Footprinting and Reconnaissance**, Sherlock was used to search for publicly available social media profiles associated with the target individual. The tool identified profile URLs across multiple platforms, allowing publicly available personal information to be gathered as part of passive reconnaissance.

---

## Advantages

- Passive reconnaissance technique
- Searches hundreds of websites simultaneously
- Open-source and free
- Fast username enumeration
- Simple command-line interface
- Useful for OSINT investigations

---

## Limitations

- Relies on publicly available profiles
- Results depend on username consistency across platforms
- Some websites may block automated requests
- False positives may occasionally occur
- Does not access private account information

---

## Best Practices

- Verify discovered profiles manually before drawing conclusions.
- Combine Sherlock results with other OSINT tools.
- Respect privacy and applicable legal requirements.
- Use only against authorized targets.
- Save results for documentation and future analysis.

---

## Used In

- Module 02 – Footprinting and Reconnaissance

---

## References

- https://github.com/sherlock-project/sherlock
- https://sherlock-project.github.io/