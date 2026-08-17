# CEH v13 Emerging Concepts & Exam Trap Addendum

> **Status:** 🎯 Active Exam Addendum  
> **Source:** Practice Test 2 Analysis & Refinement  
> **Focus:** High-Yield Traps, Modern CEH v13 Objectives & Edge Cases  

---

# 1. Cloud & Modern Attack Surface

* **Shadow IT:** Unapproved technologies, cloud storage, or SaaS applications deployed by business units without IT oversight. Detected using **CASB** (Cloud Access Security Brokers) and **EASM**.
* **EASM (External Attack Surface Management):** Continuous discovery of internet-facing assets (forgotten subdomains, shadow cloud buckets, exposed APIs) from an external adversary's viewpoint.
* **CTEM (Continuous Threat Exposure Management):** 5-stage continuous security cycle: **Scope $\rightarrow$ Discover $\rightarrow$ Prioritize $\rightarrow$ Validate $\rightarrow$ Mobilize**.
* **Attack Path Management (APM):** Graph-based risk modeling that visualizes exploit chains connecting multi-step vulnerabilities to high-value assets.
* **AWS Instance Metadata (SSRF):** Attacking `http://169.254.169.254/latest/meta-data/iam/security-credentials/[role]` via SSRF to exfiltrate IAM role session tokens. **IMDSv2** mitigates this by requiring a session token header (`X-aws-ec2-metadata-token`).
* **5G Network Slicing:** Partitioning a physical 5G network into multiple virtual slices (eMBB, URLLC, mMTC). Isolation failure risks cross-tenant data leakage.

---

# 2. Advanced Defense Evasion & Malware Mechanics

* **Process Doppelgänging:** Abuses Windows NTFS transactions (`TxF`) to run malicious code from an uncommitted transacted file, evading AV/EDR scanning memory-mapped files on disk.
* **Direct Syscalls (SysWhispers / HellsGate):** Bypasses userland EDR API hooks in `ntdll.dll` by invoking kernel system call instructions directly.
* **DLL Sideloading:** Placing a malicious DLL in the execution directory of a legitimate, digitally signed application so it inherits the trusted process context.
* **Cobalt Strike Malleable C2:** Customizing C2 network profiles to blend traffic into legitimate protocols (e.g., mimicking Google Analytics or Amazon traffic) to bypass NIDS signatures.
* **Sandbox Evasion:** Malware checking for static cursors, uptime, CPU cores ($<2$), hypervisor artifacts (CPUID), or sleep acceleration to delay execution in automated sandboxes.

---

# 3. Web Application & API Security Nuances

* **Session Riding:** The formal synonym for **Cross-Site Request Forgery (CSRF)**—riding on top of an existing, authenticated user session.
* **GraphQL Security:** Single endpoint architecture; target **Introspection** (`__schema` queries) to expose the full data model, deeply nested queries for resource exhaustion DoS, and broken object authorization on resolvers.
* **HTTP Parameter Pollution (HPP):** Passing duplicate parameters (`?user=admin&user=guest`) to exploit differences in parsing logic between front-end WAFs and backend servers.
* **HTTP Request Smuggling:** Exploits discrepancies in how front-end proxies and back-end web servers process `Content-Length` vs `Transfer-Encoding` boundaries, desynchronizing request queues.
* **OWASP ASVS (Application Security Verification Standard):** Framework providing three assurance levels (Level 1: Basic, Level 2: Standard, Level 3: Critical) for defining and testing web app security controls.
* **Mobile Security Framework (MobSF):** Automated, all-in-one framework for performing static and dynamic security analysis of Android APKs and iOS IPAs.

---

# 4. Wireless, IoT/OT, System & Cryptographic Traps

* **OWASP IoT Top 10 #1:** **Weak, Guessable, or Hardcoded Passwords** (not SQLi).
* **Airodump-ng vs. Aireplay-ng vs. Aircrack-ng:**
  * `airodump-ng`: Passive wireless sniffer and traffic recorder.
  * `aireplay-ng`: Active frame injector (transmits deauthentication frames `-0`).
  * `aircrack-ng`: Offline password cracker for captured 4-way handshakes.
* **WPS PIN Attack (Reaver / Bully):** Exploits the 8-digit WPS PIN design flaw (independent verification of the two halves reduces keyspace to 11,000 attempts).
* **KRACK (Key Reinstallation Attack):** Exploits the WPA2 4-way handshake to force cryptographic nonce reuse and key reinstallation (CVE-2017-13077).
* **POODLE (CVE-2014-3566):** Exploits SSL 3.0 CBC padding oracle via forced protocol downgrade.
* **Password Storage Standard:** Never encrypt passwords. Always use **memory-hard adaptive hashing** (Argon2id, bcrypt, scrypt) with unique random salts.
* **Ettercap / Bettercap:** Layer 2/3 MITM and ARP poisoning tools across local networks.
* **Active Directory Triumvirate Risks:** (1) MS17-010 on Domain Controller, (2) KRBTGT account unrotated $>5$ years, (3) Domain Admin accounts with SPNs (Kerberoastable).