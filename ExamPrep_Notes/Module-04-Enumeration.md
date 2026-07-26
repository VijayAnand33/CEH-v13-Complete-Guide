# Module 04: Enumeration

## 4.1 Enumeration Fundamentals & Core Protocols

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

## 4.2 NetBIOS & SMB Enumeration Mechanics

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

### Command-Line & Native Utility Syntax
* **`nbtstat` (Windows Native):**
  * `nbtstat -a <IP>` : Obtains NetBIOS name table of target remote machine by IP address.
  * `nbtstat -A <Hostname>` : Obtains NetBIOS table using target hostname.
  * `nbtstat -c` : Displays local NetBIOS name cache (IP-to-NetBIOS mappings).
  * `nbtstat -S` : Displays NetBIOS sessions table along with destination IP addresses.
* **SMB Null Session & Native Mounting (`net use`):**
  * `net use \\<Target_IP>\IPC$ "" /user:""` : Establishes an unauthenticated SMB **Null Session** over port `445` or `139` to query IPC$ shares (frequently blocked/restricted in modern Windows builds via `RestrictAnonymous` registry keys).
  * `net use Z: \\<Target_IP>\<ShareName> /user:<Username> <Password>` : Maps remote network file share to local drive letter `Z:`.

### Specialized SMB Tools
* **`enum4linux` / `enum4linux-ng`:** Perl/Python tool used to enumerate Windows/Samba data:
  * `enum4linux -U <IP>` : Extracts userlist.
  * `enum4linux -S <IP>` : Extracts shares.
  * `enum4linux -a <IP>` : Executes full enumeration (Users, Shares, Password Policies, Groups, LSA secrets).
* **`smbclient`:**
  * `smbclient -L \\<Target_IP> -N` : Lists shares anonymously (`-N` no password).
  * `smbclient \\<Target_IP>\<Share> -U "<User>%<Pass>"` : Connects directly to share.

---

## 4.3 SNMP (Simple Network Management Protocol) Enumeration

### SNMP Architecture & Operations
* **Agents & Managers:** Agents run on managed devices (routers, switches, servers); Managers issue queries to agents.
* **Community Strings (Cleartext Passwords):**
  * `public` : Default read-only community string.
  * `private` : Default read-write community string.
* **SNMP Protocol Versions:**
  * **SNMPv1 & SNMPv2c:** Pass community strings in unencrypted cleartext across UDP port 161. Vulnerable to packet sniffing and brute-forcing.
  * **SNMPv3:** Adds cryptographic authentication (MD5/SHA) and privacy encryption (DES/AES). Bypasses traditional SNMP probing attacks.

### Management Information Base (MIB) & Object Identifiers (OIDs)
* **MIB Tree Structure:** Hierarchical tree database defining managed objects.
* **Critical CEH OID Top-Level Branches:**
  * `.1.3.6.1.2.1.1` (`sysDescr`) : System Description (OS, Kernel, System uptime).
  * `.1.3.6.1.2.1.25.1.1` : System Uptime.
  * `.1.3.6.1.2.1.25.4.2.1.2` (`hrSWRunName`) : Running Processes list.
  * `.1.3.6.1.2.1.25.6.3.1.2` (`hrSWInstalledName`) : Installed Software applications.
  * `.1.3.6.1.4.1.77.1.2.25` : System User Accounts list.
  * `.1.3.6.1.4.1.77.1.2.3.1.1` : Shared Network Drives.

### SNMP Query Tools
* **`snmpwalk` / `snmpget` (Linux):**
  * `snmpwalk -v2c -c public <Target_IP> .1.3.6.1.2.1` : Recursively walks the entire MIB tree using SNMPv2c and `public` community string.
  * `snmpcheck -t <Target_IP> -c public` : Automated Perl utility extracting user accounts, interfaces, network stats, and listening ports.
* **`snmputil` (Windows CLI):**
  * `snmputil walk <Target_IP> public .1.3.6.1.2.1.1`

---

## 4.4 Active Directory & LDAP Enumeration

### Active Directory Hierarchy & LDAP Paths
* **LDAP Data Structure:** Hierarchical structure (`DC` = Domain Component, `OU` = Organizational Unit, `CN` = Common Name).
  * *Example Path:* `CN=John Doe,OU=Engineering,DC=corp,DC=local`
* **Global Catalog (Ports 3268/3269):** Distributed data repository containing a searchable subset of every object in an Active Directory forest.

### AD & LDAP Tools
* **`ldapsearch` (OpenLDAP CLI):**
  * `ldapsearch -x -h <Target_IP> -b "dc=corp,dc=local" "(objectClass=user)" sAMAccountName` : Issues anonymous or authenticated query (`-x` simple auth) to pull user account attributes (`sAMAccountName`).
* **Active Directory Explorer (`ADExplorer.exe` - Sysinternals):** Advanced GUI viewer/editor used to navigate LDAP hierarchies, inspect Schema definitions, view user property flags (e.g., `userAccountControl`, `pwdLastSet`), and take offline snapshots of AD databases for offline analysis.
* **`BloodHound` & `SharpHound`:** Automated AD domain mapping framework that collects domain relationships, ACLs, and trust paths to visualize privilege escalation routes to Domain Admin.

---

## 4.5 RPC, NFS & File System Enumeration

### Remote Procedure Call (RPC) Enumeration
* **RPC Endpoint Mapper:** Operates on TCP/UDP port `135` (Windows) and TCP/UDP port `111` (`rpcbind` / `portmap` on Linux).
* **`rpcclient` (Linux SMB/RPC Probing):**
  * `rpcclient -U "" <Target_IP>` : Binds to RPC endpoint anonymously.
  * *Internal RPC Commands:*
    * `enumdomusers` : Lists all domain users.
    * `enumdomgroups` : Lists all domain groups.
    * `queryuser <RID_Hex>` : Pulls detailed attributes for specific Relative Identifier (e.g., `0x1f4` for Administrator).
    * `netshareenumall` : Enumerates available share resources.

### Network File System (NFS) Enumeration
* **NFS Services:** Exports directories across IP networks (Ports 2049 & 111).
* **Enumeration Commands:**
  * `showmount -e <Target_IP>` : Queries the NFS daemon (`rpcbind`) to display the target's export list (exported directories and allowed client IP ranges).
  * `mount -t nfs <Target_IP>:/<RemoteShare> /mnt/local` : Mounts exported directory to local system path.
* **SuperEnum & Automated Scripts:** Combined enumeration scripts (e.g., `SuperEnum`) automate `showmount`, `rpcscan`, and `nbtscan` queries against subnets.

---

## 4.6 DNS, SMTP & Network Infrastructure Enumeration

### DNS Enumeration & Zone Transfers
* **DNS Records:** `A` (IPv4), `AAAA` (IPv6), `MX` (Mail Exchange), `NS` (Name Server), `PTR` (Reverse Lookup), `SOA` (Start of Authority), `TXT` (SPF/DKIM/DMARC attributes).
* **DNS Zone Transfer (AXFR):** Replicates DNS database across secondary servers over TCP port 53. If unmanaged, exposes all internal subdomains, hostnames, and IP mappings.
  * **`dig` Command Syntax:**
    * `dig @<DNS_Server> <Target_Domain> AXFR` : Attempts full AXFR zone transfer against target name server.
    * `dig -x <Target_IP> @<DNS_Server>` : Executes reverse PTR query.
  * **`nslookup` Interactive Zone Transfer:**
    ```text
    nslookup
    > server <Target_DNS_IP>
    > set type=any
    > ls -d <Target_Domain>