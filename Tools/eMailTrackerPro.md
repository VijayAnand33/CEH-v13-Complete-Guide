# eMailTrackerPro

## Overview

eMailTrackerPro is an email analysis and tracking tool used to examine email headers and trace the route taken by an email from the sender to the recipient. It helps identify the originating IP address, mail servers, routing path, geographical location, and WHOIS information, making it valuable for email investigations and digital forensics.

---

## Purpose

eMailTrackerPro is primarily used to:

- Analyze email headers
- Trace email routing paths
- Identify sender IP addresses
- Determine the geographical location of mail servers
- Perform Network WHOIS lookups
- Support email investigations and digital forensics

---

## Key Features

- Email header analysis
- Email route tracing
- World map visualization
- Network WHOIS lookup
- Email summary generation
- Hop-by-hop route analysis
- IP address identification

---

## Installation

eMailTrackerPro is a Windows-based application that requires installation using the provided setup wizard.

---

## Basic Usage

```text
Launch eMailTrackerPro
        ↓
Select Trace Headers
        ↓
Paste Email Header
        ↓
Trace Email
        ↓
Analyze Route
        ↓
Review WHOIS Information
```

---

## Commonly Used Features

| Feature | Description |
|---------|-------------|
| Trace Headers | Analyzes the email header and begins tracing the message. |
| Email Summary | Displays a summarized view of the traced email. |
| Route Map | Visualizes the email's routing path on a world map. |
| Hop Analysis | Displays each mail server through which the email passed. |
| Network WHOIS | Retrieves WHOIS information for discovered IP addresses. |
| My Trace Reports | Stores previously generated trace reports. |

---

## Typical Workflow

```mermaid
flowchart LR

A[Obtain Email Header]
--> B[Paste into eMailTrackerPro]

B --> C[Analyze Header]

C --> D[Trace Mail Route]

D --> E[Identify Mail Servers & IP Addresses]

E --> F[Review WHOIS & Geographic Information]

style A fill:#0d1117,color:#fff,stroke:#58a6ff
style B fill:#161b22,color:#fff,stroke:#58a6ff
style C fill:#161b22,color:#fff,stroke:#58a6ff
style D fill:#161b22,color:#fff,stroke:#58a6ff
style E fill:#161b22,color:#fff,stroke:#58a6ff
style F fill:#238636,color:#fff,stroke:#2ea043
```

---

## CEH Practical Example

In **Module 02 – Footprinting and Reconnaissance**, eMailTrackerPro was used to analyze an email header obtained from Gmail. The tool traced the email's routing path, identified intermediate mail servers, displayed the route on a world map, and performed a Network WHOIS lookup to gather additional information about the discovered IP addresses.

---

## Advantages

- Easy-to-use graphical interface
- Visualizes the email routing path
- Supports Network WHOIS lookups
- Provides hop-by-hop analysis
- Useful for email investigations
- Assists digital forensic analysis

---

## Limitations

- Depends on complete email headers
- Cannot trace emails with missing or spoofed header information accurately
- Geolocation data may not always represent the sender's actual location
- Requires Windows for execution

---

## Best Practices

- Always analyze the complete email header.
- Validate findings using additional email analysis tools.
- Understand that sender IPs may be hidden by email providers.
- Use only for authorized investigations.
- Correlate routing information with WHOIS data for better analysis.

---

## Used In

- Module 02 – Footprinting and Reconnaissance

---

## References

- https://www.emailtrackerpro.com/
- https://www.emailtrackerpro.com/help/ 