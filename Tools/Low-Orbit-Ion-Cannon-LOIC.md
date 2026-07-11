# Low Orbit Ion Cannon (LOIC)

> **Category:** Network Stress Testing Tool
>
> **Platform:** Windows
>
> **License:** Open Source
>
> **Official Website:** https://sourceforge.net/projects/loic/

---

## Overview

Low Orbit Ion Cannon (LOIC) is an open-source network stress testing tool designed to generate high volumes of TCP, UDP, or HTTP traffic toward a target system. Although originally created for network testing, it is widely known for demonstrating Denial-of-Service attack techniques in controlled laboratory environments.

---

## Key Features

- TCP flood attacks
- UDP flood attacks
- HTTP flood attacks
- Adjustable thread count
- Configurable packet rate
- Target IP and port selection
- Simple graphical interface
- Real-time traffic generation

---

## Common Usage

Typical workflow:

1. Specify target IP address
2. Select attack method
3. Configure threads and power
4. Lock onto target
5. Launch traffic generation

---

## CEH Practical Example

During **Module 10 – Denial-of-Service**, LOIC was configured on multiple attacker machines to generate UDP flood traffic against a Windows 11 target system. The generated traffic was monitored using Anti DDoS Guardian to detect abnormal network activity and demonstrate defensive mitigation techniques.

---

## Defensive Perspective

Organizations should deploy DDoS mitigation services, rate limiting, Web Application Firewalls (WAFs), intrusion prevention systems, and network traffic analysis to detect and block high-volume flooding attacks before they affect service availability.

---

## Used In

- Module 10 – Denial-of-Service

---

## References

- Official Website: https://sourceforge.net/projects/loic/
- Official Documentation: https://github.com/NewEraCracker/LOIC