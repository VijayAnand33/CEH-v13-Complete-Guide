# Cowrie

> **Category:** SSH/Telnet Honeypot
>
> **Platform:** Linux
>
> **License:** Open Source
>
> **Official Website:** https://cowrie.org/

---

## Overview

Cowrie is an open-source SSH and Telnet honeypot designed to detect, monitor, and record unauthorized access attempts. It emulates a realistic Linux environment, allowing security professionals to safely observe attacker behavior, capture brute-force attempts, record executed commands, and collect threat intelligence without exposing production systems.

---

## Key Features

- SSH and Telnet honeypot
- Brute-force attack detection
- Command logging
- Session recording
- Threat intelligence collection
- Fake Linux environment
- Attacker behavior monitoring
- Forensic log generation

---

## Common Usage

Start the Cowrie honeypot:

```bash
bin/cowrie start
```

View captured logs:

```bash
tail cowrie.log
```

---

## CEH Practical Example

During **Module 12 – Evading IDS, Firewalls, and Honeypots**, Cowrie was deployed as an SSH honeypot to capture unauthorized login attempts and attacker activity. After connecting through SSH, the attacker executed multiple Linux reconnaissance commands, all of which were recorded by Cowrie for security analysis and forensic investigation.

---

## Defensive Perspective

Organizations can deploy Cowrie to detect brute-force attacks, collect attacker intelligence, monitor unauthorized SSH activity, and improve incident response without exposing critical production systems.

---

## Used In

- Module 12 – Evading IDS, Firewalls, and Honeypots

---

## References

- Official Website: https://cowrie.org/
- Official Documentation: https://docs.cowrie.org/