# Module 17: Hacking Mobile Platforms

## Tier 1: Core Concepts & Principles

### Mobile Architecture & Attack Vectors
* **Mobile Operating System Models:** Android (Linux kernel base, open ecosystem, permission-based app sandbox) vs. iOS (Darwin Unix base, closed "walled garden" ecosystem, strict code-signing).
* **Android Application Sandbox:** Every Android app runs in its own isolated process with a unique Linux User ID (UID). Apps cannot access each other's data files without explicitly declared and user-granted permissions (declared in `AndroidManifest.xml`).
* **Android Debug Bridge (ADB) Architecture:** A versatile command-line bridge enabling communication between a host computer and an Android client/emulator.
  * *Components:* `adb client` (runs on host machine), `adb daemon (adbd)` (runs on Android device as a background process), and `adb server` (manages communication between client and daemon).
  * *Network Exposure Risk:* When ADB over Wi-Fi/TCP/IP is enabled on **Port 5555**, any unauthenticated network host can issue administrative commands to the device without physical USB access.

---

## Tier 2: Technical Analysis & Mechanics

### Mobile Malware & Remote Access Trojans (RATs)
* **Android Package (APK) Mechanics:** A compressed zip archive containing application code (`classes.dex` compiled Java/Kotlin bytecode), resources (`res/`), compiled assets, and the crucial configuration file `AndroidManifest.xml`.
* **Reverse Connection Trojans (AndroRAT / MSFvenom):**
  1. *Payload Binding/Generation:* A malicious payload is embedded inside an APK file configured to initiate an outbound TCP socket connection back to an attacker-controlled listener IP/port.
  2. *Execution & Permission Abuses:* Upon execution, the payload requests broad permissions (`READ_SMS`, `CAMERA`, `RECORD_AUDIO`, `READ_CONTACTS`, `ACCESS_FINE_LOCATION`).
  3. *Remote Command Execution:* Once connected, the attacker can silently dump SMS logs, capture screenshots, stream camera/microphone feeds, and access device shell environments.
* **ADB Exploitation Mechanics (PhoneSploit-Pro / ADB CLI):**
  * Attackers scan networks for open TCP Port 5555.
  * Upon connecting via `adb connect <Target_IP>:5555`, the attacker gains direct shell access (`adb shell`), allowing them to install unverified APKs (`adb install`), pull internal app databases (`adb pull`), or capture live screens (`adb exec-out screencap -p`).

---

## Tier 3: CLI, Tools & Framework Syntax

### Core Mobile Assessment Utilities
* **Android Debug Bridge (`adb`):** Native Google CLI tool for Android device management, file transfers, and debugging.
* **PhoneSploit-Pro:** Automated exploitation framework leveraging raw ADB commands over TCP/IP to automate Android device compromise.
* **AndroRAT:** Python/Java-based Remote Access Tool designed to generate reverse-shell Android payloads.
* **APKTool:** Reverse engineering utility used to decompile APKs into readable Smali code and recompile modified application archives.

### Key Commands & Execution Syntax

```bash
# Connect to Remote Android Device via ADB over TCP/IP (Port 5555)
adb connect <Target_IP>:5555

# List All Connected ADB Devices and Status
adb devices

# Spawn Interactive Remote Linux Shell on Target Android Device
adb shell

# Install Malicious APK File Remotely onto Target Device
adb install /tmp/malicious_payload.apk

# Pull Internal App Files/Database from Device to Host
adb pull /sdcard/Download/sensitive_data.db /tmp/looted_db/

# Push File from Host to Target Android File System
adb push /tmp/exploit_script.sh /data/local/tmp/

# Capture Live Remote Device Screenshot and Save to Local Machine
adb exec-out screencap -p > /tmp/target_screenshot.png

# Enumerate All Installed Application Packages on Target
adb shell pm list packages -f

# PhoneSploit-Pro Automated Framework Launch
python3 phonesploitpro.py

# Decompile APK File for Code Review using APKTool
apktool d /tmp/target_app.apk -o /tmp/decompiled_app/
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Mobile Platform Hardening & Defense Architecture
* **Disable Network Debugging:** Disable ADB over TCP/IP in Developer Options; restrict ADB connections strictly to physical USB interfaces with host RSA fingerprint verification enabled.
* **Android Application Security Standards:**
  * Disallow installation of applications from unknown sources (`Settings -> Security -> Install Unknown Apps = Disabled`).
  * Enforce Google Play Protect and integrate mobile Endpoint Detection and Response (EDR) / Antivirus tools (e.g., AVG, Lookout, CrowdStrike Mobile) to perform real-time signature and behavioral scanning.
* **Application Hardening & Code Signing:**
  * Developers must enforce **Root Detection** and **SafetyNet / Play Integrity API** checks to prevent apps from running on compromised/jailbroken devices.
  * Enable Code Obfuscation (ProGuard / R8) to hinder reverse engineering via APKTool or JADX.
  * Restrict `AndroidManifest.xml` debuggable flag (`android:debuggable="false"`).
* **Enterprise Mobile Management (MDM / EMM):** Enforce containerization (e.g., Samsung Knox, Work Profiles) to separate personal data from enterprise assets, enforce remote wipe capabilities, and mandate strict passcodes.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **ADB TCP Port 5555** | Default port for unauthenticated Android Debug Bridge network connections | Allows full remote shell access without physical USB connection |
| **AndroidManifest.xml** | Core configuration file containing app permissions, components, and hardware requirements | Primary target during mobile app reverse engineering to discover permission abuses |
| **`classes.dex`** | Compiled Java/Kotlin bytecode file executed by the Android Runtime (ART/Dalvik) | Decompiled using tools like JADX or Dex2Jar to inspect source code |
| **PhoneSploit-Pro** | Automated CLI tool that exploits exposed network ADB services (Port 5555) | Uses ADB protocol commands under the hood to control target phones |
| **AndroRAT** | Python-based tool to generate malicious reverse-shell Android payloads | Embeds persistent background services to exfiltrate SMS, call logs, and location |
| **APKTool** | Utility used to decompile, disassemble, and recompile APK files | Converts binary APKs into editable Smali code and raw XML assets |
| **Android Sandbox** | Linux UID-based process isolation separating apps from one another | Prevents App A from reading App B's internal data directory `/data/data/<package>` |
| **Google Play Protect** | Built-in Android malware scanner analyzing app behaviors | Blocks installation of known malicious APKs and flags suspicious background activity |
| **`adb exec-out screencap`** | Command to remotely capture target device screen | Directly streams screen PNG bytes to host terminal output |
| **Jailbreaking vs. Rooting** | Rooting = Gaining privilege escalation on Android; Jailbreaking = Bypassing iOS code-signing restrictions | Both eliminate OS sandbox protections, increasing device vulnerability |