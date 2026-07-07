# Traceroute

## Overview

Traceroute is a network diagnostic utility used to determine the path that packets travel from a source system to a destination host. It identifies the intermediate routers (hops) along the route and measures the time taken to reach each hop, making it an essential tool for network troubleshooting and reconnaissance.

On Windows, the utility is available as **tracert**, while Linux and Unix-based systems use **traceroute**. Although the command names differ, both perform the same fundamental function.

---

## Purpose

Traceroute is primarily used to:

- Discover the network path to a target host
- Identify intermediate routers (hops)
- Analyze network topology
- Detect routing issues
- Measure network latency
- Support network reconnaissance

---

## Key Features

- Hop-by-hop route discovery
- Network latency measurement
- Cross-platform availability
- Network path visualization
- Router identification
- Supports IPv4 and IPv6

---

## Installation

### Windows

`tracert` is included by default with Windows.

### Linux

`traceroute` may require installation.

```bash
sudo apt install traceroute
```

---

## Basic Syntax

### Windows

```cmd
tracert <target>

tracert -h <max_hops> <target>

tracert /?
```

### Linux

```bash
traceroute <target>

traceroute -m <max_hops> <target>
```

---

## Commonly Used Options

| Command / Option | Description |
|------------------|-------------|
| `tracert <target>` | Performs traceroute on Windows. |
| `traceroute <target>` | Performs traceroute on Linux. |
| `-h <max_hops>` | Sets the maximum number of hops (Windows). |
| `-m <max_hops>` | Sets the maximum number of hops (Linux). |
| `/?` | Displays Windows command help. |
| `--help` | Displays Linux command help. |

---

## Typical Workflow

```mermaid
flowchart LR

A[Specify Target Host]
--> B[Send Probe Packets]

B --> C[Identify Intermediate Routers]

C --> D[Measure Response Time]

D --> E[Map Network Path]

E --> F[Analyze Network Topology]

style A fill:#0d1117,color:#fff,stroke:#58a6ff
style B fill:#161b22,color:#fff,stroke:#58a6ff
style C fill:#161b22,color:#fff,stroke:#58a6ff
style D fill:#161b22,color:#fff,stroke:#58a6ff
style E fill:#161b22,color:#fff,stroke:#58a6ff
style F fill:#238636,color:#fff,stroke:#2ea043
```

---

## CEH Practical Example

In **Module 02 – Footprinting and Reconnaissance**, **tracert** was used on Windows and **traceroute** was used on the Parrot Security machine to identify the network path between the source and the target domain. The utilities revealed intermediate routers, hop counts, and response times, enabling analysis of the target's network topology and routing path.

---

## Advantages

- Included by default on most operating systems
- Simple and easy to use
- Identifies intermediate routers
- Measures network latency
- Useful for troubleshooting and reconnaissance
- Cross-platform support

---

## Limitations

- Some routers block or filter traceroute packets
- Firewalls may prevent accurate results
- Results may vary depending on routing changes
- Does not reveal internal network architecture

---

## Best Practices

- Compare results from different locations when possible.
- Use with Ping and nslookup for comprehensive network analysis.
- Analyze unusual latency or routing behavior.
- Use only against authorized targets.
- Treat routing information as reconnaissance intelligence rather than definitive network architecture.

---

## Used In

- Module 02 – Footprinting and Reconnaissance

---

## References

- https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tracert
- https://man7.org/linux/man-pages/man8/traceroute.8.html
- https://www.ibm.com/docs/en/aix/7.3?topic=t-traceroute-command