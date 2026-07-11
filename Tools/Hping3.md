# Hping3

> **Category:** Network Packet Generator
>
> **Platform:** Linux
>
> **License:** Open Source
>
> **Official Website:** https://www.hping.org/

---

## Overview

Hping3 is an advanced command-line network packet generator and analyzer used for security testing, firewall evaluation, network troubleshooting, and penetration testing. It enables users to craft and transmit custom TCP, UDP, ICMP, and RAW IP packets while analyzing responses from target systems.

---

## Key Features

- Custom TCP, UDP, ICMP, and RAW packet generation
- SYN, ACK, FIN, and UDP flood testing
- Firewall and IDS testing
- Network latency analysis
- Packet fragmentation
- TTL manipulation
- Host discovery
- Scriptable command-line interface

---

## Common Usage

Send TCP SYN packets:

```bash
sudo hping3 -S <Target-IP> -p 80
```

Common workflow:

1. Select protocol
2. Configure packet parameters
3. Specify destination
4. Send crafted packets
5. Analyze responses

---

## CEH Practical Example

During **Module 10 – Denial-of-Service**, Hping3 was used to generate large volumes of custom network packets toward a target system to demonstrate how packet flooding techniques can exhaust system resources and affect service availability during controlled DoS testing.

---

## Defensive Perspective

Administrators should implement firewall filtering, rate limiting, IDS/IPS detection rules, and traffic monitoring to identify abnormal packet generation. Continuous monitoring helps detect packet flooding attempts before they significantly impact network availability.

---

## Used In

- Module 10 – Denial-of-Service

---

## References

- Official Website: https://www.hping.org/
- Official Documentation: http://www.hping.org/wiki-sub/