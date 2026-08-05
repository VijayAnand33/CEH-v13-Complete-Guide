# Module 04: Enumeration

## Tier 1: Core Concepts & Principles
* **Enumeration Phase:** The 3rd phase of ethical hacking (following Reconnaissance and Scanning). Involves establishing active connections to target systems and executing directed queries to extract user accounts, group names, shares, system banners, routing tables, and service configurations.
* **Enumeration vs. Reconnaissance/Scanning:**
  * *Reconnaissance:* Passive or non-intrusive target mapping.
  * *Scanning:* Identifying live hosts, open ports, and service types.
  * *Enumeration:* Intrusion-adjacent active queries against open ports to reveal specific resource identifiers, internal usernames, Active Directory trees, and system parameters.
* **Target Protocol Vectors & Listening Ports:**
  * **NetBIOS Name Service:** Port `137` (UDP)
  * **NetBIOS Session Service / SMB:** Ports `139` (TCP) & `445` (TCP)
  * **SNMP (Simple Network Management Protocol):** Port `161` (UDP Probe) & Port `162` (UDP Trap)
  * **LDAP / LDAPS:** Port `389` (TCP/UDP) / Port `636` (SSL/TLS TCP)
  * **Global Catalog (Active Directory):** Port `3268` (TCP) / Port `3269` (SSL TCP)
  * **RPC / Portmapper:** Port `111` (TCP/UDP) & Port `135` (TCP)
  * **DNS Zone Transfers:** Port `53` (TCP)
  * **SMTP:** Ports `25`, `465`, `587` (TCP)
  * **NFS (Network File System):** Ports `2049` (TCP/UDP) & `111` (RPC)

---

## Tier 2: Technical Analysis & Mechanics

### NetBIOS Suffix Identifiers (Hexadecimal Codes)
NetBIOS names consist of 16 characters (15 ASCII characters + 1 hex byte suffix designating system role).

| Hex Suffix | Name Type | System Role / Description |
| :--- | :--- | :--- |
| **`<00>`** | Workstation | NetBIOS Workstation Service (Machine name or Domain) |
| **`<03>`** | Messenger Service | NetBIOS Messenger Service (User or Computer name) |
| **`<20>`** | Server Service | NetBIOS File Server / SMB File Sharing enabled |
| **`<1B>`** | Master Browser | Domain Master Browser (PBD / Domain Controller) |
| **`<1C>`** | Domain Group | Domain Controller Group (Responds to logon requests) |
| **`<1D>`** | Master Browser | Local Master Browser for subnet |
| **`<1E>`** | Group Name | Browser Service Elections |

### SNMP Architecture, Community Strings & OIDs
* **Agents & Managers:** Agents run on managed devices (routers, switches, servers); Managers issue queries to agents.
* **Community Strings (Cleartext Passwords):**
  * `public` : Default read-only community string.
  * `private` : Default read-write community string.
* **SNMP Protocol Versions:**
  * **SNMPv1 & SNMPv2c:** Pass community strings in unencrypted cleartext across UDP port 161. Vulnerable to packet sniffing and brute-forcing.
  * **SNMPv3:** Adds cryptographic authentication (MD5/SHA) and privacy encryption (DES/AES). Bypasses traditional SNMP probing attacks.
* **MIB Tree Structure & Critical CEH OID Top-Level Branches:**
  * `.1.3.6.1.2.1.1` (`sysDescr`) : System Description (OS, Kernel, System uptime).
  * `.1.3.6.1.2.1.25.1.1` : System Uptime.
  * `.1.3.6.1.2.1.25.4.2.1.2` (`hrSWRunName`) : Running Processes list.
  * `.1.3.6.1.2.1.25.6.3.1.2` (`hrSWInstalledName`) : Installed Software applications.
  * `.1.3.6.1.4.1.77.1.2.25` : System User Accounts list.
  * `.1.3.6.1.4.1.77.1.2.3.1.1` : Shared Network Drives.

### Active Directory Hierarchy & LDAP Paths
* **LDAP Data Structure:** Hierarchical structure (`DC` = Domain Component, `OU` = Organizational Unit, `CN` = Common Name).
  * *Example Path:* `CN=John Doe,OU=Engineering,DC=corp,DC=local`
* **Global Catalog (Ports 3268/3269):** Distributed data repository containing a searchable subset of every object in an Active Directory forest.

### RPC & NFS Mechanics
* **RPC Endpoint Mapper:** Operates on TCP/UDP port `135` (Windows) and TCP/UDP port `111` (`rpcbind` / `portmap` on Linux).
* **Network File System (NFS):** Exports directories across IP networks (Ports 2049 & 111).

### DNS Records & Zone Transfers
* **DNS Records:** `A` (IPv4), `AAAA` (IPv6), `MX` (Mail Exchange), `NS` (Name Server), `PTR` (Reverse Lookup), `SOA` (Start of Authority), `TXT` (SPF/DKIM/DMARC attributes).
* **DNS Zone Transfer (AXFR):** Replicates DNS database across secondary servers over TCP port 53. If unmanaged, exposes all internal subdomains, hostnames, and IP mappings.

---

## Tier 3: CLI, Tools & Framework Syntax

### NetBIOS & SMB Utilities
* **`nbtstat` (Windows Native):**
  * `nbtstat -a <IP>` : Obtains NetBIOS name table of target remote machine by IP address.
  * `nbtstat -A <Hostname>` : Obtains NetBIOS table using target hostname.
  * `nbtstat -c` : Displays local NetBIOS name cache (IP-to-NetBIOS mappings).
  * `nbtstat -S` : Displays NetBIOS sessions table along with destination IP addresses.
* **SMB Null Session & Native Mounting (`net use`):**
  * `net use \\<Target_IP>\IPC$ "" /user:""` : Establishes an unauthenticated SMB **Null Session** over port `445` or `139` to query IPC$ shares (frequently blocked/restricted in modern Windows builds via `RestrictAnonymous` registry keys).
  * `net use Z: \\<Target_IP>\<ShareName> /user:<Username> <Password>` : Maps remote network file share to local drive letter `Z:`.
* **Specialized SMB Tools:**
  * `enum4linux -U <IP>` : Extracts userlist.
  * `enum4linux -S <IP>` : Extracts shares.
  * `enum4linux -a <IP>` : Executes full enumeration (Users, Shares, Password Policies, Groups, LSA secrets).
  * `smbclient -L \\<Target_IP> -N` : Lists shares anonymously (`-N` no password).
  * `smbclient \\<Target_IP>\<Share> -U "<User>%<Pass>"` : Connects directly to share.

### SNMP Query Tools
* **`snmpwalk` / `snmpget` (Linux):**
  * `snmpwalk -v2c -c public <Target_IP> .1.3.6.1.2.1` : Recursively walks the entire MIB tree using SNMPv2c and `public` community string.
  * `snmpcheck -t <Target_IP> -c public` : Automated Perl utility extracting user accounts, interfaces, network stats, and listening ports.
* **`snmputil` (Windows CLI):**
  * `snmputil walk <Target_IP> public .1.3.6.1.2.1.1`

### AD & LDAP Tools
* **`ldapsearch` (OpenLDAP CLI):**
  * `ldapsearch -x -h <Target_IP> -b "dc=corp,dc=local" "(objectClass=user)" sAMAccountName` : Issues anonymous or authenticated query (`-x` simple auth) to pull user account attributes (`sAMAccountName`).
* **Active Directory Explorer (`ADExplorer.exe` - Sysinternals):** Advanced GUI viewer/editor used to navigate LDAP hierarchies, inspect Schema definitions, view user property flags (e.g., `userAccountControl`, `pwdLastSet`), and take offline snapshots of AD databases for offline analysis.
* **`BloodHound` & `SharpHound`:** Automated AD domain mapping framework that collects domain relationships, ACLs, and trust paths to visualize privilege escalation routes to Domain Admin.

### RPC, NFS & DNS Tooling Commands
* **`rpcclient` (Linux SMB/RPC Probing):**
  * `rpcclient -U "" <Target_IP>` : Binds to RPC endpoint anonymously.
  * `enumdomusers` : Lists all domain users.
  * `enumdomgroups` : Lists all domain groups.
  * `queryuser <RID_Hex>` : Pulls detailed attributes for specific Relative Identifier (e.g., `0x1f4` for Administrator).
  * `netshareenumall` : Enumerates available share resources.
* **NFS Commands:**
  * `showmount -e <Target_IP>` : Queries the NFS daemon (`rpcbind`) to display the target's export list (exported directories and allowed client IP ranges).
  * `mount -t nfs <Target_IP>:/<RemoteShare> /mnt/local` : Mounts exported directory to local system path.
  * *SuperEnum:* Combined enumeration script automating `showmount`, `rpcscan`, and `nbtscan` queries.
* **DNS Query Syntax:**
  * `dig @<DNS_Server> <Target_Domain> AXFR` : Attempts full AXFR zone transfer against target name server.
  * `dig -x <Target_IP> @<DNS_Server>` : Executes reverse PTR query.
  * `nslookup` Interactive Zone Transfer:
    ```text
    nslookup
    > server <Target_DNS_IP>
    > set type=any
    > ls -d <Target_Domain>
    ```

---

## Tier 4: Real-World Countermeasures & Defense
* **SMB / NetBIOS Protection:** Disable SMBv1, enforce SMB Signing, restrict null sessions via `RestrictAnonymous` registry keys, and block ports 137-139 / 445 at edge perimeter firewalls.
* **SNMP Hardening:** Upgrade from SNMPv1/v2c to SNMPv3 (enforcing encryption and cryptographic auth), change default community strings (`public`, `private`), and restrict SNMP access to administrative IP ranges via ACLs.
* **Active Directory & LDAP Defense:** Disable anonymous LDAP binds, enforce LDAP signing/channel binding, restrict permissions on AD objects, and monitor for abnormal query volume via BloodHound activity detections.
* **DNS & NFS Security:** Restrict DNS Zone Transfers (AXFR) strictly to authorized secondary DNS server IP addresses, restrict NFS share exports via exact IP whitelist rules in `/etc/exports`, and block unauthorized RPC portmapper (111/135) connections.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **Enumeration Definition** | Phase 3; active queries against open ports | Extracts user account lists, share names, AD trees, and banners |
| **NetBIOS `<00>`** | Workstation / Host Name designation | Machine or domain name |
| **NetBIOS `<20>`** | File Server / Server Service | Indicates SMB file sharing is enabled on target |
| **NetBIOS `<1B>`** | Domain Master Browser | Identifies Primary Domain Controller / Master Browser |
| **`nbtstat -a <IP>`** | Queries remote NetBIOS name table by IP | Uses uppercase `-A` for querying by Hostname |
| **`nbtstat -c`** | Local NetBIOS cache display | Shows IP-to-NetBIOS mappings held locally |
| **SMB Null Session** | `net use \\<IP>\IPC$ "" /user:""` | Unauthenticated connection over port 445/139 to extract shares/users |
| **`enum4linux -a`** | Comprehensive Linux SMB enumeration | Runs full automated check for users, shares, groups, and policies |
| **SNMP Port 161 / 162** | UDP Port 161 (Queries) / UDP Port 162 (Traps) | 161 handles manager-to-agent queries; 162 handles agent traps |
| **Community Strings** | `public` (Read-only) / `private` (Read-write) | Act as cleartext passwords in SNMPv1 and SNMPv2c |
| **SNMPv3** | Secure SNMP protocol version | Adds MD5/SHA authentication and DES/AES encryption |
| **OID `.1.3.6.1.2.1.1`** | `sysDescr` top-level branch | Extracts OS version, system description, and uptime |
| **OID `.1.3.6.1.4.1.77.1.2.25`** | System User Accounts list | Queries list of local target usernames via SNMP |
| **`snmpwalk`** | Recursively walks MIB tree | Issues continuous GETNEXT requests using community string |
| **LDAP Port 389 / 636** | LDAP (389 TCP/UDP) / LDAPS (636 SSL TCP) | Directory queries for Active Directory objects |
| **Global Catalog Ports** | TCP 3268 / SSL TCP 3269 | AD forest-wide searchable directory service repository |
| **`ADExplorer.exe`** | Sysinternals AD snapshot tool | Navigates LDAP hierarchy and takes offline AD snapshots |
| **`BloodHound` / `SharpHound`**| Visualizes AD trust paths and ACLs | Identifies privilege escalation routes to Domain Admin |
| **RPC Endpoint Mapper** | Port 135 (Windows) / Port 111 (Linux `rpcbind`) | Maps RPC services to dynamic listening ports |
| **`rpcclient` (`queryuser`)** | Queries specific Relative Identifier (RID) | `queryuser 0x1f4` targets default Administrator account |
| **`showmount -e <IP>`** | Queries NFS daemon exports | Displays list of exported directories and allowed client IPs |
| **DNS Zone Transfer (AXFR)** | Replicates full DNS database over TCP Port 53 | Exposes all subdomains, hostnames, and IP mappings via `dig AXFR` |

---

## Common Enumeration Services & Ports

| Port | Protocol | Service | Full Form | Primary Purpose | CEH Enumeration Focus |
|------:|:--------:|---------|-----------|-----------------|-----------------------|
| **53** | TCP | DNS | **Domain Name System** | Resolves domain names to IP addresses. TCP is primarily used for DNS Zone Transfers (AXFR). | DNS Enumeration, Zone Transfer |
| **111** | TCP/UDP | RPC / Portmapper | **Remote Procedure Call / Portmapper (rpcbind)** | Maps RPC services to dynamically assigned ports on Linux/Unix systems. | NFS Enumeration, RPC Enumeration |
| **135** | TCP | RPC | **Remote Procedure Call** | Enables communication between Windows services and applications. | Windows RPC Enumeration |
| **137** | UDP | NBNS | **NetBIOS Name Service** | Resolves NetBIOS computer names to IP addresses within a local network. | NetBIOS Enumeration, LLMNR/NBT-NS Poisoning |
| **139** | TCP | NetBIOS Session Service | **NetBIOS Session Service** | Legacy Windows file and printer sharing over NetBIOS. | SMB Enumeration (Legacy) |
| **161** | UDP | SNMP | **Simple Network Management Protocol** | Retrieves management information from network devices such as routers, switches, and servers. | SNMP Enumeration |
| **162** | UDP | SNMP Trap | **Simple Network Management Protocol Trap** | Sends alerts and event notifications from managed devices to the SNMP manager. | Identify monitoring infrastructure |
| **389** | TCP/UDP | LDAP | **Lightweight Directory Access Protocol** | Provides access to directory services such as Microsoft Active Directory. | LDAP Enumeration |
| **445** | TCP | SMB | **Server Message Block** | Native Windows file, printer, and resource sharing without NetBIOS. | SMB Enumeration, Pass-the-Hash, EternalBlue |
| **465** | TCP | SMTPS | **Simple Mail Transfer Protocol Secure** | Secure email transfer using SSL/TLS (legacy). | SMTP Enumeration |
| **587** | TCP | SMTP Submission | **Simple Mail Transfer Protocol** | Authenticated email submission from mail clients to mail servers. | SMTP Enumeration |
| **636** | TCP | LDAPS | **Lightweight Directory Access Protocol Secure** | Encrypted LDAP communication using SSL/TLS. | Secure LDAP Enumeration |
| **2049** | TCP/UDP | NFS | **Network File System** | Allows Linux/Unix systems to share files and directories across a network. | NFS Enumeration |
| **3268** | TCP | Global Catalog | **Active Directory Global Catalog** | Searches directory objects across the entire Active Directory forest. | Active Directory Enumeration |
| **3269** | TCP | Global Catalog SSL | **Secure Active Directory Global Catalog** | Encrypted Global Catalog service over SSL/TLS. | Secure Active Directory Enumeration |

---

## Service Abbreviations

| Abbreviation | Full Form |
|--------------|-----------|
| **DNS** | Domain Name System |
| **RPC** | Remote Procedure Call |
| **NBNS** | NetBIOS Name Service |
| **SMB** | Server Message Block |
| **SNMP** | Simple Network Management Protocol |
| **LDAP** | Lightweight Directory Access Protocol |
| **LDAPS** | Lightweight Directory Access Protocol Secure |
| **NFS** | Network File System |
| **SMTP** | Simple Mail Transfer Protocol |
| **SMTPS** | Simple Mail Transfer Protocol Secure |
| **AD** | Active Directory |
| **GC** | Global Catalog |
| **AXFR** | Authoritative Zone Transfer |
| **LLMNR** | Link-Local Multicast Name Resolution |
| **NBT-NS** | NetBIOS Name Service |
| **RDP** | Remote Desktop Protocol |
| **TGS** | Ticket Granting Service |
| **TGT** | Ticket Granting Ticket |

---

## CEH Quick Memory Notes

| Port | Remember This |
|------|---------------|
| **53/TCP** | DNS Zone Transfer (AXFR) |
| **111** | Linux RPC / Portmapper |
| **135** | Windows RPC |
| **137** | NetBIOS Name Resolution |
| **139** | Legacy SMB over NetBIOS |
| **161** | SNMP Query (Manager → Device) |
| **162** | SNMP Trap (Device → Manager) |
| **389** | LDAP (Active Directory) |
| **445** | Modern SMB |
| **465** | Legacy Secure SMTP |
| **587** | Modern Authenticated SMTP |
| **636** | Secure LDAP (LDAPS) |
| **2049** | NFS File Sharing |
| **3268** | Active Directory Global Catalog |
| **3269** | Secure Global Catalog |

---

## CEH Exam Tips

- **53/TCP** → Attempt **DNS Zone Transfer (AXFR)**.
- **111** → Enumerate **NFS** and RPC services.
- **135** → Enumerate Windows RPC services.
- **137** → Enumerate NetBIOS names and perform **NBT-NS/LLMNR poisoning**.
- **139 / 445** → Enumerate SMB shares, users, groups, and OS information.
- **161** → Dump device configuration using SNMP if community strings are weak.
- **389 / 636** → Enumerate Active Directory users, groups, computers, and policies.
- **2049** → Enumerate exported NFS shares.
- **3268 / 3269** → Query objects across the entire Active Directory forest.
- **25 / 465 / 587** → Perform SMTP user enumeration and mail server assessment.