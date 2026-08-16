# CEH v13 Master Exam Summary & Rapid Recall Guide

> **Status:** 🎯 Ready for Exam  
> **Modules Covered:** Modules 01 to 20  
> **Target Certification:** EC-Council Certified Ethical Hacker (CEH v13)  

---

# 1. Master Ports & Protocols Cheat Sheet

| Port | Protocol / Service | Security Implications & CEH Exam Triggers |
| :--- | :--- | :--- |
| **20 / 21** | FTP (File Transfer Protocol) | Cleartext credentials; Port 21 = Control connection, Port 20 = Active data transfer; Anonymous login risk |
| **22** | SSH / SFTP | Secure remote administration; target for brute-force attacks (`hydra -l user -P pass.txt ssh://target`) |
| **23** | Telnet | Legacy unencrypted remote terminal; sniffable credentials |
| **25** | SMTP | Simple Mail Transfer Protocol; `VRFY` and `EXPN` commands abused for username enumeration |
| **53** | DNS (Domain Name System) | UDP 53 = Standard queries; **TCP 53 = DNS Zone Transfers (`AXFR`)**; Target for DNS Amplification |
| **67 / 68** | DHCP (Dynamic Host Configuration) | Port 67 = Server, Port 68 = Client; Target for **DHCP Starvation** and Rogue DHCP attacks |
| **69** | TFTP (Trivial FTP) | UDP-based; no authentication; commonly leveraged in embedded device/firmware attacks |
| **80 / 443** | HTTP / HTTPS | Web traffic; Port 80 = Cleartext HTTP; Port 443 = Encrypted TLS/HTTPS |
| **88** | Kerberos | Windows Domain Authentication; AS-REQ, AS-REP, TGS-REQ, TGS-REP; Target for **Kerberoasting** & **Golden/Silver Tickets** |
| **102** | Siemens S7 Communication | Default industrial communication port for **Siemens S7 PLCs** (ICS/OT) |
| **110 / 995** | POP3 / POP3S | Post Office Protocol; Port 110 = Cleartext, Port 995 = TLS encrypted |
| **123** | NTP (Network Time Protocol) | UDP 123; `monlist` query abused for **NTP Amplification / Reflection DoS** |
| **135** | MS RPC Endpoint Mapper | Windows RPC locator; queried by enumeration tools (e.g., `rpcdump`) |
| **137 / 138 / 139** | NetBIOS | Port 137 = Name Service (UDP), Port 138 = Datagram, Port 139 = Session Service (TCP); target of **Responder** LLMNR/NBT-NS poisoning |
| **143 / 993** | IMAP / IMAPS | Internet Message Access Protocol; Port 143 = Cleartext, Port 993 = TLS encrypted |
| **161 / 162** | SNMP | Port 161 = SNMP Agent (queries), Port 162 = SNMP Trap; Community strings `public` (read) / `private` (write); SNMPv3 adds USM/VACM security |
| **389 / 636** | LDAP / LDAPS | Lightweight Directory Access Protocol; Port 389 = Cleartext, Port 636 = TLS; Base DN searches |
| **445** | SMB (Server Message Block) | Direct host SMB; EternalBlue (MS17-010), PsExec, Pass-the-Hash, null sessions (`IPC$`) |
| **502** | Modbus TCP | Legacy industrial SCADA/OT protocol; **completely unauthenticated and unencrypted** |
| **514** | Syslog | UDP 514 standard unencrypted system event logging |
| **1433** | Microsoft SQL Server (MSSQL) | Default database port; target of SQLi, `xp_cmdshell` execution, and database enumeration |
| **1521** | Oracle Database | Default TNS listener port for Oracle DB |
| **1883 / 8883** | MQTT (IoT Messaging) | Port 1883 = Cleartext MQTT (subscribe to `#` for wildcard capture); Port 8883 = MQTTS (TLS encrypted) |
| **2375 / 2376** | Docker Daemon Engine | Port 2375 = Unencrypted REST API (remote root container breakout risk); Port 2376 = TLS protected |
| **3306** | MySQL Database | Default MySQL database port |
| **3389** | RDP (Remote Desktop Protocol) | BlueKeep (CVE-2019-0708) vulnerability; target of automated brute-force attacks |
| **5555** | Android Debug Bridge (ADB) | Default TCP port for wireless ADB debugging (`adb connect target:5555` $\rightarrow$ `adb shell`) |
| **8080 / 8443** | HTTP Alternate / Proxies | Common web proxy, staging web app, and alternative web management ports (Burp Suite, Tomcat) |

---

# 2. Master Tool-to-Phase Mapping & Primary Commands

### Reconnaissance, OSINT & Scanning (Modules 1–5)
* **theHarvester:** OSINT email, domain, subdomain, employee, and PGP key harvester.
  * `theHarvester -d targetcompany.com -b all`
* **Amass / Sublist3r:** Subdomain enumeration utilizing active DNS brute-forcing and passive OSINT archives.
* **Nmap:** Network exploration and security auditing port scanner.
  * `-sS`: TCP SYN Stealth Scan (Half-open; sends `SYN` $\rightarrow$ receives `SYN/ACK` $\rightarrow$ responds with `RST`).
  * `-sT`: Full TCP Connect Scan (completes 3-way handshake; leaves application logs).
  * `-sU`: UDP Scan (slow; open/filtered ports drop packets, closed ports return ICMP Port Unreachable Type 3 Code 3).
  * `-sN / -sF / -sX`: Null (`0x00`), FIN (`0x01`), Xmas (FIN/PSH/URG flags set); open ports drop, closed ports reply `RST` (RFC 793).
  * `-sA`: ACK Scan; used to map firewall rulesets (`RST` = unfiltered, No Response/ICMP = filtered).
  * `-sV -sC`: Version detection with default NSE scripts.
  * `-O`: Remote OS fingerprinting via TCP/IP stack behavior.
  * `-D RND:10`: Decoy scan mixing real scanner IP with 10 random spoofed addresses.
* **Hping3:** Custom TCP/IP packet crafting utility for firewall testing, traceroute, and raw DoS testing.
  * `hping3 -S 192.168.1.1 -p 80 -c 5` (SYN probe)
  * `hping3 --flood -S -p 80 192.168.1.1` (SYN Flood DoS)

### Enumeration & Vulnerability Assessment (Modules 4–5)
* **nbtstat:** Windows NetBIOS utility. Record types: `<00>` = Workstation/Domain, `<20>` = File Server Service, `<1B>` = Domain Master Browser.
  * `nbtstat -A 192.168.1.50`
* **enum4linux / enum4linux-ng:** SMB, NetBIOS, RPC, and SID enumeration script against Windows/Samba targets.
* **snmpwalk:** Automated SNMP tree crawler.
  * `snmpwalk -v2c -c public 192.168.1.1`
* **OpenVAS / Nessus:** Comprehensive automated vulnerability assessment scanners mapping findings to CVEs and CVSS ratings.

### System Hacking, Privilege Escalation & Persistence (Module 6)
* **Responder:** LLMNR, NBT-NS, and mDNS poisoner used to capture NetNTLMv1/NetNTLMv2 hashes over the local network.
  * `responder -I eth0 -dwv`
* **Mimikatz:** Post-exploitation tool interacting with `lsass.exe` to extract plaintext credentials, Kerberos tickets, and NTLM hashes.
  * `privilege::debug` $\rightarrow$ `sekurlsa::logonpasswords`
* **Hashcat:** GPU-accelerated hash recovery engine.
  * `-m 1000`: NTLM hashes
  * `-m 5600`: NetNTLMv2 hashes
  * `-m 1800`: SHA-512 crypt Linux hashes (`$6$`)
  * `-m 22000`: WPA/WPA2/WPA3 PMKID / Handshake captures
* **John the Ripper:** CPU password cracker (`john --wordlist=rockyou.txt hashes.txt`).

### Sniffing, MITM & Defense Evasion (Modules 8 & 11)
* **Wireshark / TShark:** GUI/CLI packet analysis engines.
* **Ettercap / Bettercap:** Dynamic ARP cache poisoning and Man-in-the-Middle (MITM) attack platforms.
* **Macof:** Tool for flooding switch CAM tables with random MAC addresses to force a fail-open (hub-mode) state.

### Web Server & Application Hacking (Modules 12–14)
* **Nikto:** Web server scanner detecting outdated software versions, vulnerable CGI binaries, and configuration errors.
  * `nikto -h http://target.com`
* **Burp Suite:** Web application assessment proxy intercepting, modifying, and analyzing HTTP/HTTPS requests.
* **SQLmap:** Automated SQL injection discovery and exploitation engine.
  * `sqlmap -u "http://target.com/item?id=1" --dbs` (enumerate databases)
  * `sqlmap -u "http://target.com/item?id=1" --os-shell` (interactive OS command shell)
* **WPScan:** WordPress CMS vulnerability scanner for plugins, themes, and users.

### Wireless Hacking (Module 15)
* **Aircrack-ng Suite:**
  * `airmon-ng start wlan0`: Enables monitor mode (`wlan0mon`).
  * `airodump-ng -c <channel> --bssid <BSSID> -w capture wlan0mon`: Captures raw 802.11 frames to capture the 4-way handshake.
  * `aireplay-ng -0 5 -a <BSSID> -c <Client_MAC> wlan0mon`: Transmits 5 802.11 deauthentication frames to force client re-association.
  * `aircrack-ng -w wordlist.txt -b <BSSID> capture-01.cap`: Cracks WPA/WPA2 Pre-Shared Keys offline.

### Mobile, IoT/OT, Cloud & Cryptography (Modules 16–20)
* **APKTool:** Decompiles and rebuilds Android `.apk` files into human-readable resources and Smali bytecode (`apktool d app.apk`).
* **can-utils (candump / canplayer / cansend):**
  * `candump -l vcan0`: Captures and logs raw CAN bus frames.
  * `canplayer -I candump.log`: Replays recorded CAN traffic to execute replay attacks.
  * `cansend vcan0 123#DEADBEEF`: Injects raw 11-bit CAN frames onto the bus.
* **AADInternals:** PowerShell module for Azure Active Directory / Entra ID reconnaissance and tenant enumeration.
  * `Invoke-AADIntReconAsOutsider -DomainName target.com`
* **AWS CLI (`aws`):**
  * `aws s3 ls s3://<bucket>/ --no-sign-request`: Lists S3 bucket contents without authentication.
  * `aws iam attach-user-policy --user-name lowpriv --policy-arn arn:aws:iam::aws:policy/AdministratorAccess`: IAM privilege escalation.
* **Trivy:** Static vulnerability scanner for Docker container images, filesystems, and repository dependencies (`trivy image nginx:latest`).
* **VeraCrypt:** Full disk and virtual volume encryption using AES-XTS mode.
* **CyberChef:** Web-based Swiss Army knife for multi-layer hashing, Base64 decoding, hex manipulation, and cryptographic analysis.

---

# 3. Top High-Yield CEH Exam Decision Rules & Rapid Triggers

```
[Target: Network/Switch Layer]
├── CAM Table Overflow ────────► Macof (floods switch memory to force broadcast/hub mode)
├── MITM / Traffic Redirection ─► ARP Spoofing (forged ARP replies with attacker MAC)
└── Switch Defense ─────────────► Dynamic ARP Inspection (DAI) + DHCP Snooping

[Target: Windows / Active Directory]
├── Extract Plaintext / Tickets ► Mimikatz (reads memory of lsass.exe)
├── Steal NetNTLMv2 Hash ───────► Responder (poisons LLMNR / NBT-NS broadcasts)
├── Forged Kerberos TGT ────────► Golden Ticket (requires compromised krbtgt NTLM hash)
└── Forged Kerberos TGS ────────► Silver Ticket (requires specific service account NTLM hash)

[Target: Web Applications & Databases]
├── User-to-User Data Access ──► IDOR / BOLA (manipulating sequential ID parameters)
├── Execute OS Commands via DB ─► xp_cmdshell (Microsoft SQL Server extended procedure)
├── Client-side Script Execution► XSS (Reflected = transient, Stored = persistent in DB)
└── SQL Injection Defense ──────► Parameterized Queries / Prepared Statements (bind variables)

[Target: Wireless, IoT, Cloud & Mobile]
├── Force WPA2 Handshake Capture► aireplay-ng -0 (Deauth frames)
├── Decompile Android APK ──────► APKTool (extracts AndroidManifest.xml and Smali code)
├── Industrial Protocol (OT) ───► Modbus TCP (Port 502, completely unauthenticated)
├── Unauthenticated S3 Access ──► aws s3 ls s3://bucket/ --no-sign-request
└── Container Breakout Risk ────► Running containers with --privileged or mounting /var/run/docker.sock

[Target: Cryptography]
├── Confidentiality (Bulk) ─────► Symmetric Encryption (AES: 128-bit block, 128/192/256-bit keys)
├── Insecure Block Cipher Mode ─► ECB (Electronic Codebook - identical blocks yield identical ciphertext)
├── Key Exchange Over Untrusted ─► Diffie-Hellman (DH / ECDH)
├── Digital Signatures (Non-Rep)─► Sender encrypts hash with Private Key; Receiver verifies with Public Key
└── Password Storage Defense ───► Salting + Key Stretching (Argon2, bcrypt, PBKDF2)
```