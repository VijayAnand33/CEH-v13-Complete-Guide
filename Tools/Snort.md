# Snort

> **Category:** Network Intrusion Detection System (NIDS)
>
> **Platform:** Windows, Linux, macOS
>
> **License:** Open Source
>
> **Official Website:** https://www.snort.org/

---

## Overview

Snort is an open-source Network Intrusion Detection System (NIDS) capable of performing real-time traffic analysis, packet logging, and intrusion detection. It uses rule-based signature matching to identify malicious network activities such as port scans, denial-of-service attacks, protocol exploits, and reconnaissance attempts. Snort is widely used by security professionals to monitor network traffic and generate alerts for suspicious behavior.

---

## Key Features

- Real-time packet analysis
- Signature-based intrusion detection
- Protocol analysis
- Packet logging
- Custom detection rules
- Network traffic monitoring
- Alert generation
- Network intrusion prevention support

---

## Common Usage

List available network interfaces:

```bash
snort -W
```

Start Snort in IDS mode:

```bash
snort -i2 -A console -c C:\Snort\etc\snort.conf -l C:\Snort\log -K ascii
```

---

## CEH Practical Example

During **Module 12 – Evading IDS, Firewalls, and Honeypots**, Snort was configured as a Network Intrusion Detection System (NIDS) to monitor incoming ICMP traffic. When the attacker machine launched continuous ping requests against the target, Snort detected the intrusion, generated real-time alerts, and recorded the attack within its IDS log files.

---

## Defensive Perspective

Organizations should deploy Snort with updated detection rules, continuously monitor generated alerts, regularly review IDS logs, and integrate the platform with SIEM solutions to improve threat detection, incident response, and network visibility.

---

## Used In

- Module 12 – Evading IDS, Firewalls, and Honeypots

---

## References

- Official Website: https://www.snort.org/
- Official Documentation: https://docs.snort.org/