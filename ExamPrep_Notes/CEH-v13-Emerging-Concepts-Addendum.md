# CEH v13 Emerging Concepts & Master Exam Addendum

> **Status:** 🎯 Consolidated Exam Addendum (Post-Tests 2 & 3 Review)  
> **Source:** Practice Tests 2 & 3, Diagnostic Drills, and Scenario Analysis  
> **Focus:** Modern Attack Surface, Evasion, Cloud, AD Infrastructure, Protocols & Edge Cases  

---

# 1. Cloud & Modern Attack Surface

* **Shadow IT:** Unapproved technologies, cloud storage, or SaaS applications deployed by business units without IT oversight. Detected using **CASB** (Cloud Access Security Brokers) and **EASM**.
* **EASM (External Attack Surface Management):** Continuous discovery of internet-facing assets (forgotten subdomains, shadow cloud buckets, exposed APIs) from an external adversary's viewpoint.
* **CTEM (Continuous Threat Exposure Management):** Gartner's 5-stage continuous security cycle: **Scope $\rightarrow$ Discover $\rightarrow$ Prioritize $\rightarrow$ Validate $\rightarrow$ Mobilize**.
* **Attack Path Management (APM):** Graph-based risk modeling that visualizes exploit chains connecting multi-step vulnerabilities to high-value assets (e.g., Domain Controllers).
* **AWS Instance Metadata (SSRF):** Attacking `http://169.254.169.254/latest/meta-data/iam/security-credentials/[role]` via SSRF to exfiltrate IAM role session tokens. **IMDSv2** mitigates this by requiring a session token header (`X-aws-ec2-metadata-token`).
* **AWS Security Token Service (STS) Abuse:** Attackers with limited IAM access abuse `sts:AssumeRole` to escalate privileges horizontally or vertically across cloud accounts.
* **AWS S3 Dangerous Misconfigurations:**
  * `Principal: *` + `Action: s3:GetObject` $\rightarrow$ Unauthenticated public read exposure.
  * `Action: s3:PutObject` for any principal $\rightarrow$ Arbitrary file upload, malware hosting under trusted corporate domains, and dependency package poisoning.
* **5G Network Slicing:** Partitioning a physical 5G network into multiple virtual slices (eMBB, URLLC, mMTC). Isolation failure risks cross-tenant data leakage.
* **CNAPP (Cloud-Native Application Protection Platform):** Integrated cloud security suite combining **CSPM** (posture/misconfigs), **CWPP** (workload/runtime defense), and **CIEM** (identity/excessive entitlements).
* **Cloud-Native Ransomware:** Targets cloud backup infrastructure directly—deleting S3 versioning/snapshots and disabling recovery vaults rather than just encrypting local files.

---

# 2. Advanced Defense Evasion & Malware Mechanics

* **Process Doppelgänging:** Abuses Windows NTFS transactions (`TxF`) to run malicious code from an uncommitted transacted file, evading AV/EDR scanning memory-mapped files on disk.
* **Direct Syscalls (SysWhispers / HellsGate):** Bypasses userland EDR API hooks in `ntdll.dll` by invoking kernel system call instructions directly.
* **DLL Sideloading / Search Order Hijacking:** Placing a malicious DLL in the execution directory of a legitimate, digitally signed application so it inherits the trusted process context.
* **Cobalt Strike Malleable C2:** Customizing C2 network profiles to blend traffic into legitimate protocols (e.g., mimicking Google Analytics, Amazon, or jQuery CDN traffic) to bypass NIDS signatures.
* **Sandbox Evasion:** Malware checking for static cursors, uptime, CPU cores ($<2$), hypervisor artifacts (CPUID), or sleep acceleration to delay execution in automated sandboxes.
* **BYOVD (Bring Your Own Vulnerable Driver):** Loading a legitimate, digitally signed but vulnerable third-party driver to execute kernel-mode code and disable EDR sensors (mitigated by HVCI).
* **Windows Credential Guard:** Uses Virtualization-Based Security (VBS) to isolate credential secrets (NTLM hashes, Kerberos tickets) inside the `LSAIso` process, completely blocking `Mimikatz sekurlsa` dumping from memory.
* **Timestomping:** Altering MAC (Modified, Accessed, Created) file timestamps via tools like `timestomp` or `touch -t` to blend webshells into old filesystem timelines (countered by `$MFT` forensic analysis).
* **Canary Tokens:** Deception artifacts (bogus Word docs, AWS API keys, unique internal URLs) placed across environments that trigger instant alerts when accessed by adversaries.

---

# 3. Web Application, API & Infrastructure Attacks

* **Penetration Testing Lifecycle Flow:**
  $$\text{Initial Low-Privilege Shell (e.g., } \texttt{www-data}\text{)} \longrightarrow \mathbf{\text{Local Privilege Escalation (root / SYSTEM)}} \longrightarrow \text{Internal Discovery / Lateral Movement}$$
* **Session Riding:** The formal synonym for **Cross-Site Request Forgery (CSRF)**—riding on top of an existing, authenticated user session.
* **Session Token Storage Vulnerability:** Storing authentication tokens in `localStorage` makes them permanently accessible to any injected JavaScript (**XSS Token Theft**). Session tokens should reside in `HttpOnly; Secure; SameSite=Strict` cookies or memory.
* **GraphQL Security & Reconnaissance:**
  * **Introspection:** Querying `{ __schema { types { name } } }` dumps the complete backend schema, queries, and mutations.
  * **Attacks:** Deeply nested queries (Resource Exhaustion / DoS) and broken object authorization on field resolvers.
* **HTTP Parameter Pollution (HPP):** Passing duplicate parameters (`?user=admin&user=guest`) to exploit differences in parsing logic between front-end WAFs and backend servers.
* **HTTP Request Smuggling:** Exploits discrepancies in how front-end proxies and back-end web servers process `Content-Length` vs `Transfer-Encoding` boundaries, desynchronizing request queues.
* **JWT Algorithm Confusion Attacks:**
  * **`alg: none` Bypass:** Stripping the signature and changing the header to `"alg": "none"`.
  * **RS256 $\rightarrow$ HS256 Confusion:** Switching from asymmetric (RS256) to symmetric (HS256) and signing the token with the server's public key as the HMAC secret.
* **API Authorization Failures:**
  * **BOLA (Broken Object Level Authorization / IDOR - API1):** Manipulating object references (e.g., changing `/api/users/1001` to `/api/users/1002`) to access another user's data.
  * **BFLA (Broken Function Level Authorization - API5):** Standard user invoking administrative endpoints (e.g., accessing `/api/v1/admin/deleteUser`).
* **Soft 404 Handling (Gobuster / FFUF):** When applications return `HTTP 200 OK` for nonexistent pages, use `--wildcard` in Gobuster or response size filtering (`-fs <size>`) in FFUF.
* **Spring Boot Actuator Exposure:** Unsecured `/actuator/env` exposes system environment variables and passwords; `/actuator/heapdump` leaks raw JVM memory and encryption keys.
* **SAML Injection / XML Signature Wrapping (XSW):** Manipulating XML signatures in SAML assertions so that identity providers validate the original signature while processing modified attacker identity claims.
* **CORS Misconfiguration:** An API responding with `Access-Control-Allow-Origin: https://attacker.com` and `Access-Control-Allow-Credentials: true` allows malicious sites to read authenticated API responses.
* **OWASP ASVS (Application Security Verification Standard):** Defines web app security verification across 3 assurance tiers: Level 1 (Basic), Level 2 (Standard), Level 3 (Critical).
* **Mobile Tooling:**
  * **MobSF (Mobile Security Framework):** Automated static code auditing and dynamic sandbox scanning of APK/IPA files.
  * **Frida:** Dynamic runtime instrumentation engine for function hooking in memory (bypassing SSL pinning and root detection).

---

# 4. Active Directory, System Hacking & Network Infrastructure

* **The 4 Primary Windows Persistence Mechanisms:**
  1. Registry `Run` Keys (`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`)
  2. Scheduled Tasks (`schtasks /create`)
  3. Malicious Services (`sc create`)
  4. DLL Search Order Hijacking / DLL Sideloading
* **Active Directory High-Impact Attacks:**
  * **Golden Ticket:** Forged Kerberos TGT using the compromised **`krbtgt` NTLM hash** (full domain dominance, valid up to 10 years; requires rotating `krbtgt` password twice).
  * **Silver Ticket:** Forged Kerberos TGS using the specific **service account's NTLM hash** (stealthy access to one service; bypasses the KDC).
  * **Kerberoasting:** Requesting TGS tickets for accounts with Service Principal Names (SPNs) and cracking them offline (`hashcat -m 13100`).
  * **Kerberos Pre-Auth Brute Force (`kerbrute`):** Sends AS-REQ packets directly to the KDC without completing full login; generates **Event ID 4771** (Kerberos pre-auth failed) instead of Event ID 4625 (NTLM failure).
  * **AD CS ESC1:** Misconfigured certificate templates with `ENROLLEE_SUPPLIES_SUBJECT` and Client Authentication EKU allow any domain user to request certificates as a Domain Admin.
  * **Resource-Based Constrained Delegation (RBCD):** Attacker with write access (`GenericWrite`) on a computer object sets `msDS-AllowedToActOnBehalfOfOtherIdentity` to impersonate domain users.
  * **Unconstrained Delegation Abuse:** Stores connecting users' TGTs in server memory; coercing domain controllers (via PrinterBug / SpoolSample) extracts the DC's TGT from memory.
* **Linux Privilege Escalation via Sudo (GTFOBins):** Exploiting `NOPASSWD` entries for editors/utilities:
  * `sudo vim -c ':!/bin/bash'`
  * `sudo find / -exec /bin/sh \;`
  * `sudo less /etc/profile` $\rightarrow$ `!/bin/sh`
* **High-Yield Post-Exploitation Commands:**
  * Active Listening Ports: `ss -tulnp` or `netstat -tulnp`
  * Active Established C2 Connections: `netstat -an | grep ESTABLISHED`
  * Local Subnet ARP Cache: `cat /proc/net/arp` or `arp -a`
  * Unquoted Service Path Discovery: `wmic service get name,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"`

---

# 5. Network Protocols, Default Ports & Cryptography

* **High-Yield Network Ports:**
  * **Port 502 (Modbus TCP):** Industrial ICS/SCADA protocol lacking native authentication/encryption; allows direct register read/write.
  * **Port 4444 (Meterpreter Reverse TCP):** Standard default payload listener port for Metasploit C2 sessions.
  * **Port 5985 / 5986 (WinRM / WS-Management):** Port 5985 is cleartext HTTP (NTLM interception risk); Port 5986 uses HTTPS.
  * **Port 6379 (Redis Unauthenticated RCE):** Open Redis allows arbitrary file write via `CONFIG SET dir /root/.ssh/` and `CONFIG SET dbfilename authorized_keys`.
  * **DNS over HTTPS (DoH - Port 443/TCP):** Encrypts DNS queries inside HTTPS, bypassing enterprise DNS inspection, logging, and sinkholes.
* **DNS Rebinding & DNS-SD:** Exploits short TTLs to rebind external domains to internal IPs (`192.168.1.1`). Leveraging **mDNS / DNS-SD** (`224.0.0.251:5353/UDP`) allows attackers to discover and interact with local network services directly from a browser.
* **DNSSEC Record Types:**
  * **NSEC (Next SECure):** Validates non-existence of records but allows complete zone enumeration (**Zone Walking**).
  * **NSEC3:** Prevents zone walking by hashing non-existent record names.
* **Cryptographic Strength Equivalence (NIST):**
  $$\text{128-bit Symmetric (AES)} \approx \text{3072-bit Asymmetric (RSA/DH)} \approx \text{256-bit Elliptic Curve (ECC)}$$
* **Cryptographic Attack Specifics:**
  * **POODLE (CVE-2014-3566):** Exploits SSL 3.0 CBC padding oracle via forced protocol downgrade.
  * **KRACK (CVE-2017-13077):** Exploits the WPA2 4-way handshake to force cryptographic nonce reuse and key reinstallation.
  * **Password Storage Standard:** Never encrypt passwords. Always use **memory-hard adaptive hashing** (Argon2id, bcrypt, scrypt) with unique per-user salts.
* **High-Profile Incident Case Study:**
  * **SolarWinds Orion (SUNBURST - 2020):** Exemplified a **Software Supply Chain Attack** where attackers breached the vendor build pipeline to distribute backdoored updates downstream at mass scale.