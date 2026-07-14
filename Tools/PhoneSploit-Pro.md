# PhoneSploit-Pro

## Overview

PhoneSploit-Pro is an open-source Android penetration testing framework that leverages Android Debug Bridge (ADB) to remotely manage and interact with Android devices. It automates various ADB operations, allowing ethical hackers and penetration testers to assess the security of Android devices through an interactive command-line interface.

---

## Key Features

- Connects to Android devices using ADB.
- Captures device screenshots remotely.
- Enumerates installed applications.
- Accesses the Android device shell.
- Retrieves device information.
- Installs and uninstalls APK files.
- Performs screen recording and other remote management tasks.

---

## Common Usage

```bash
python3 phonesploitpro.py
```

---

## Practical Example

During **Module 17 – Hacking Mobile Platforms**, PhoneSploit-Pro was used to establish an ADB connection with an Android device over TCP port 5555. After connecting successfully, screenshots were captured, installed applications were enumerated, the device shell was accessed, and system information was collected to assess the security of the Android platform.

---

## Defensive Perspective

Android Debug Bridge (ADB) should be disabled when not required and should never be exposed over untrusted networks. Organizations should restrict debugging access, enforce secure device configurations, and monitor Android devices for unauthorized ADB connections.

---

## Used In

- Module 17 – Hacking Mobile Platforms

---

## Official Resources

- https://github.com/AzeemIdrisi/PhoneSploit-Pro

---