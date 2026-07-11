# PuTTY

> **Category:** SSH and Telnet Client
>
> **Platform:** Windows
>
> **License:** Open Source
>
> **Official Website:** https://www.putty.org/

---

## Overview

PuTTY is a free and open-source terminal emulator and network client that supports SSH, Telnet, Serial, and other communication protocols. It is widely used by system administrators, network engineers, and penetration testers to securely connect to remote systems, execute commands, and manage network devices. PuTTY is one of the most popular SSH clients for Windows environments.

---

## Key Features

- SSH client
- Telnet client
- Serial console support
- Secure remote administration
- Public key authentication
- Session management
- Terminal emulation
- SCP and SFTP support (via companion tools)

---

## Common Usage

Connect to a remote SSH server:

1. Launch PuTTY.
2. Enter the target IP address or hostname.
3. Select **SSH** as the connection type.
4. Click **Open**.
5. Authenticate using valid credentials.

---

## CEH Practical Example

During **Module 12 – Evading IDS, Firewalls, and Honeypots**, PuTTY was used to establish an SSH connection to the Cowrie honeypot deployed on the Ubuntu machine. After connecting, multiple Linux reconnaissance commands were executed, allowing Cowrie to capture and log the attacker's activities for security analysis and forensic investigation.

---

## Defensive Perspective

Organizations should restrict SSH access to trusted users, enforce strong authentication mechanisms such as SSH keys and multi-factor authentication, monitor login attempts, and deploy honeypots like Cowrie to detect and analyze unauthorized SSH activity.

---

## Used In

- Module 12 – Evading IDS, Firewalls, and Honeypots

---

## References

- Official Website: https://www.putty.org/
- Official Documentation: https://documentation.help/PuTTY/