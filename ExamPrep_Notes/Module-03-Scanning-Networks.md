# Module 03: Scanning Networks

## Tier 1: Core Concepts & Principles
* **Scanning Phase:** The 2nd phase of ethical hacking (preceded by Reconnaissance, followed by Enumeration). Used to identify active live hosts, open ports, running services, OS versions, and vulnerabilities.
* **Types of Scanning:**
  * **Network Scanning:** Locating active hosts on a network subnet.
  * **Port Scanning:** Probing open/closed/filtered TCP & UDP listening ports.
  * **Vulnerability Scanning:** Scanning target services against catalogs of known CVE weaknesses (e.g., Nessus, Qualys, OpenVAS, Nmap `--script vuln`).
* **TCP Control Flags (Header Fields):**
  * `SYN` (Synchronize): Initiates connection.
  * `ACK` (Acknowledge): Confirms packet receipt.
  * `FIN` (Finish): Gracefully closes connection.
  * `RST` (Reset): Abruptly aborts/rejects connection.
  * `PSH` (Push): Instructs system to pass buffer data immediately to application layer.
  * `URG` (Urgent): Marks data payload as high-priority.

---

## Tier 2: Technical Analysis & Mechanics

### Layer 2 & Layer 3 Live Host Discovery
* **ARP Ping Scan (`nmap -sn -PR`):** Probes hosts on the local Layer 2 broadcast domain. Bypasses host-based firewalls because ARP traffic cannot be blocked without breaking local communication.
* **ICMP Echo Ping Sweep (`nmap -sn -PE`):** Sends ICMP Type 8 (Echo Request). Live hosts respond with ICMP Type 0 (Echo Reply). Frequently blocked by edge firewalls.
* **ICMP Timestamp Ping (`nmap -sn -PP`):** Sends ICMP Type 13 (Timestamp Request) to obtain target clock times. Bypasses firewalls filtering Type 8 Echo requests.
* **ICMP Address Mask Ping (`nmap -sn -PM`):** Sends ICMP Type 17 (Address Mask Request) to query target subnets.
* **TCP SYN Ping (`nmap -sn -PS<ports>`):** Sends `SYN` to default port 80. Open or closed ports return `ACK` or `RST` respectively, confirming host existence.
* **TCP ACK Ping (`nmap -sn -PA<ports>`):** Sends `ACK` to port 80. Unfiltered hosts reply with `RST`.
* **UDP Ping Sweep (`nmap -sn -PU<ports>`):** Transmits UDP probes to high/closed ports. Live hosts respond with ICMP Type 3 Code 3 (*Port Unreachable*).
* **IP Protocol Scan (`nmap -sO`):** Iterates through IP protocol headers (ICMP=1, IGMP=2, TCP=6, UDP=17) to discover supported transport protocols.
* **IPv6 Host Discovery:** Traditional scanning of large 64-bit IPv6 subnets ($2^{64}$ addresses) via brute-force is infeasible. Attackers discover IPv6 addresses via email headers, Usenet, DNS logs, or by probing the link-local all-hosts multicast address `FF02::1` once a single local host is compromised.

### Advanced Port Scanning Methods & Flag Mechanics

#### Comprehensive Scan Matrix

| Scan Name | Nmap Flag | Packet Flow & Flags Set | Open Port Response | Closed Port Response | Filtered / Firewall Response |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TCP Connect Scan** | `-sT` | Full 3-Way Handshake (`SYN` $\rightarrow$ `SYN/ACK` $\rightarrow$ `ACK` $\rightarrow$ `RST`). Non-root execution. | `SYN/ACK` | `RST` | Timeout / ICMP Unreachable |
| **Stealth / SYN Scan** | `-sS` | Half-Open (`SYN` $\rightarrow$ `SYN/ACK` $\rightarrow$ `RST`). Requires root/admin. | `SYN/ACK` | `RST` | Timeout / ICMP Unreachable |
| **Inverse TCP (FIN)** | `-sF` | Only `FIN` flag set. | No Response | `RST` | ICMP Type 3 error |
| **Inverse TCP (NULL)** | `-sN` | All control flags turned OFF (`0`). | No Response | `RST` | ICMP Type 3 error |
| **Xmas Scan** | `-sX` | `FIN`, `URG`, `PSH` flags set simultaneously ("lit up like a Christmas tree"). | No Response | `RST` | ICMP Type 3 error |
| **ACK Scan** | `-sA` | `ACK` flag set. Tests firewall statefulness (does not detect open ports). | `RST` (*Unfiltered*) | `RST` (*Unfiltered*) | No Response (*Filtered*) |
| **TCP Window Scan** | `-sW` | Identical packet to ACK scan, but examines TCP Window Size field in returned `RST` packets. | `RST` with positive Window Size ($>0$) | `RST` with zero Window Size ($0$) | No Response |
| **Maimon Scan** | `-sM` | Sets `FIN/ACK` flags. Named after Uriel Maimon. | No Response | `RST` | ICMP Type 3 error |
| **IDLE / Zombie Scan** | `-sI <zombie_ip>` | Blind scan technique exploiting IP ID sequence predictability of an idle third-party host ("zombie"). | Zombie IP ID increments by **2** | Zombie IP ID increments by **1** | Zombie IP ID increments by **1** |
| **UDP Scan** | `-sU` | UDP packet sent to target port. Connectionless protocol. | UDP payload / No Response | ICMP Type 3 Code 3 (*Port Unreachable*) | ICMP Type 3 Codes 1, 2, 9, 10, 13 |

> **RFC 793 Rule:** FIN, NULL, and Xmas scans rely on RFC 793 behavior where closed ports reply with `RST` and open ports drop unexpected packets. **Windows systems violate RFC 793** and return `RST` for all ports, making inverse scans ineffective against Windows targets.

### Service & OS Fingerprinting (Banner Grabbing)
* **Active Banner Grabbing (`nmap -sV` / Telnet / Netcat / `hping3`):** Probes open ports with specific payload requests to elicit detailed service banner strings, software names, and version numbers.
* **Passive Banner Grabbing:** Captures and analyzes network packets in transit (e.g., via Wireshark) without sending probe packets directly to the target system.
* **Active OS Fingerprinting (`nmap -O`):** Probes target systems with illegal or non-standard TCP/UDP packets and analyzes TCP Window Size, IP TTL, Don't Fragment (DF) flags, and Explicit Congestion Notification (ECN) behavior against a signature database.
* **Aggressive Scan (`nmap -A`):** Executes OS detection (`-O`), Service Version detection (`-sV`), Script scanning (`-sC`), and Traceroute (`--traceroute`) in a single command.

---

## Tier 3: Tools, Syntax & Evasion Techniques

### Firewall Evasion, Obfuscation & Anonymization
* **Packet Fragmentation (`nmap -f` or `--mtu <bytes>`):** Breaks IP headers across smaller 8-byte fragment blocks to bypass stateless packet filters incapable of packet reassembly.
* **IP Decoy Scanning (`nmap -D RND:10,ME`):** Mixes spoofed IP addresses alongside the attacker's true IP address to clutter target log files.
* **Source Port Manipulation (`nmap --source-port 53` or `-g 80`):** Forces Nmap to originate probe traffic from trusted service ports (e.g., DNS `53` or HTTP `80`) to bypass restrictive inbound security rules.
* **MAC Address Spoofing (`nmap --spoof-mac 0`):** Spoofs the Ethernet hardware address to evade network-level access control lists (ACLs).
* **Firewalking:** A technique used to analyze ACL rules on firewalls by sending TCP/UDP packets with a TTL value equal to the target firewall hop plus 1. If allowed through the firewall, the packet expires at the internal router, returning an `ICMP Type 11 Code 0` (*Time-to-Live Exceeded in Transit*) message.

### Proxies, Anonymizers & Proxy Chains
* **Proxy Server:** An intermediary server acting on behalf of a client.
  * *Forward Proxy:* Used by internal clients to reach the Internet securely.
  * *Reverse Proxy:* Sits in front of internal servers to provide load balancing and security.
* **Proxy Chains (`proxychains`):** Routes traffic sequentially through multiple HTTP, SOCKS4, or SOCKS5 proxy servers to mask origin IP addresses.
* **Tor (The Onion Router):** Routes encrypted traffic through a decentralized network of volunteer-operated relays (Entry Node $\rightarrow$ Middle Relay $\rightarrow$ Exit Node) to provide online anonymity.

### Tooling Reference Summary
* **Nmap:** Primary open-source network scanner supporting host discovery, port scanning, OS detection, NSE scripts, and evasion.
* **Hping3:** Command-line packet generator and analyzer capable of sending custom TCP/IP packets:
  * *SYN Scan:* `hping3 -S <IP> -p 80`
  * *ACK Scan:* `hping3 -A <IP> -p 80`
  * *FIN/PUSH/URG Scan:* `hping3 -F -P -U <IP> -p 80`
* **NetScanTools Pro:** Commercial Windows GUI tool combining 40+ network utility modules.
* **Nessus / Qualys / OpenVAS:** Automated vulnerability scanners used to identify known software vulnerabilities and misconfigurations.
* **Metasploit Auxiliary Modules:** Scanning modules like `auxiliary/scanner/portscan/syn` and `auxiliary/scanner/discovery/arp_sweep`.
* **ShellGPT (`sgpt`):** AI CLI tool used to generate scanning syntax and parse scan results.

---

## Tier 4: Real-World Countermeasures & Defense
* **Firewall Rules:** Configure stateful inspection firewalls to drop unsolicited TCP probe packets and enforce strict egress filtering.
* **IDS/IPS Signatures:** Deploy intrusion detection rules (e.g., Snort) to flag abnormal TCP flag combinations (`FIN+PSH+URG` Xmas, `NULL` flag scans, high-frequency port sweeps).
* **System Hardening:** Disable unnecessary network services, hide service versions in HTTP/SMTP server headers (banner scrubbing), and tune TCP/IP stacks to resist idle scanning.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **ARP Ping Scan (`-PR`)** | Local Layer 2 broadcast domain discovery | Bypasses host firewalls; ARP cannot be blocked on local LAN |
| **ICMP Echo Ping (`-PE`)** | Sends ICMP Type 8 (Echo Request) | Responds with ICMP Type 0 (Reply); frequently blocked by edge firewalls |
| **ICMP Timestamp Ping (`-PP`)** | Sends ICMP Type 13 (Timestamp Request) | Obtains target clock time; bypasses Type 8 ICMP filters |
| **ICMP Address Mask Ping (`-PM`)** | Sends ICMP Type 17 | Queries target subnet masks |
| **UDP Ping Sweep (`-PU`)** | Probes high/closed UDP ports | Live host returns **ICMP Type 3 Code 3** (*Port Unreachable*) |
| **TCP Connect Scan (`-sT`)** | Completes full 3-Way Handshake | Non-root execution; easily logged by target system |
| **Stealth / SYN Scan (`-sS`)** | Half-open scan (`SYN` $\rightarrow$ `SYN/ACK` $\rightarrow$ `RST`) | Default scan; requires root/admin privileges |
| **Inverse FIN Scan (`-sF`)** | Only `FIN` flag set | Open port = No Response; Closed port = `RST` |
| **Inverse NULL Scan (`-sN`)** | All control flags turned OFF (`0`) | Open port = No Response; Closed port = `RST` |
| **Xmas Scan (`-sX`)** | Flags set: `FIN`, `URG`, `PSH` | "Lit up like a Christmas tree"; Open = No Response, Closed = `RST` |
| **ACK Scan (`-sA`)** | Tests firewall statefulness | Unfiltered returns `RST`; Filtered gives No Response |
| **TCP Window Scan (`-sW`)** | Examines TCP Window Size in returned `RST` | Positive Window ($>0$) = Open; Zero Window ($0$) = Closed |
| **Maimon Scan (`-sM`)** | Flags set: `FIN/ACK` | Open port = No Response; Closed port = `RST` |
| **IDLE / Zombie Scan (`-sI`)** | Blind scan via idle host's predictable IP ID | Open port increments Zombie IP ID by **2**; Closed/Filtered by **1** |
| **UDP Scan (`-sU`)** | Probes connectionless services | Closed port returns **ICMP Type 3 Code 3** (*Port Unreachable*) |
| **Windows RFC 793 Violation** | Inverse scans (`-sF`, `-sN`, `-sX`) fail | Windows returns `RST` for all ports regardless of state |
| **Active OS Fingerprinting (`-O`)** | Analyzes TTL, TCP Window Size, DF flags | Probes target using non-standard packets against signature DB |
| **Aggressive Scan (`-A`)** | Runs 4 distinct operations in 1 command | Combines OS detection (`-O`), Service Version (`-sV`), Scripts (`-sC`), Traceroute |
| **Packet Fragmentation (`-f`)** | Breaks IP headers into 8-byte blocks | Bypasses stateless packet filters incapable of packet reassembly |
| **Decoy Scan (`-D`)** | Mixes real IP with fake source IPs (`RND:10,ME`) | Clutters target log files |
| **Source Port Manipulation (`-g`)** | Forces traffic from DNS port `53` or HTTP `80` | Bypasses restrictive inbound rules filtering by source port |
| **Firewalking** | Probe firewall ACL rules via TTL + 1 | Packet expires at internal router returning **ICMP Type 11 Code 0** |
| **IPv6 Host Discovery** | $2^{64}$ address space makes brute-force infeasible | Uses email headers, DNS logs, or link-local multicast `FF02::1` |
| **Hping3 Custom Packets** | CLI packet generator/analyzer | `hping3 -S` (SYN), `hping3 -A` (ACK), `hping3 -F -P -U` (Xmas) |