# Module 08 — Sniffing


## Tier 1: Theoretical Underpinnings

### 1. Sniffing Fundamentals & Network Architecture

```
                       [ NETWORK ARCHITECTURE SNIFFING ]
                                       │
            ┌──────────────────────────┴──────────────────────────┐
            ▼                                                     ▼
   [ HUB-BASED NETWORK ]                                 [ SWITCH-BASED NETWORK ]
    (Shared Medium)                                       (Isolated Collision Domains)
            │                                                     │
            ▼                                                     ▼
    PASSIVE SNIFFING                                       ACTIVE SNIFFING
- Promiscuous Mode inspects                         - Requires Frame/Packet Injection
  all broadcasted frames.                             - Exploits Layer 2/3 Protocols
- Zero packet injection required.                     - Forces Switch into Fail-Open/Hub Mode.
```

**Passive Sniffing (Hubs & SPAN Ports)**
Operates without injecting packets or altering network traffic flow.
* **Hub Behavior:** Acts as a physical-layer repeater. All frames entering one port are broadcast to all other ports regardless of the destination MAC address.
* **Promiscuous Mode:** Network Interface Card (NIC) driver setting that disables standard hardware destination MAC filtering, passing all frames captured on the wire directly to the OS kernel and packet capture engine (libpcap/WinPcap/Npcap).
* **Switched Port Analyzer (SPAN) / Port Mirroring:** Administrative capability on managed switches that duplicates traffic passing through designated source ports/VLANs and forwards it to a destination port connected to an IDS/Sniffer.

**Active Sniffing (Switched Networks)**
Necessary on switch-based networks where Layer 2 frames are unicast exclusively to the destination host's switch port via the Content Addressable Memory (CAM) table. Active sniffing involves injecting frames to poison, flood, or manipulate Layer 2/3 mapping.

---

### 2. Layer 2 Attacks & CAM Table Exploitation

```
                   [ CAM TABLE FLOODING & FAIL-OPEN STATE ]
                   
  [ Attacker ] ──( 100,000+ Fake MAC/IP Frames / macof )──► [ Switch CAM Table ]
                                                                   │
                                                                   ▼
                                                          (CAM Memory Exhausted)
                                                                   │
                                                                   ▼
  [ Switch Transits to Fail-Open / Hub Mode ] ◄────────────────────┘
  (Unicast traffic broadcasted to ALL ports)
```

**CAM Table Mechanics & MAC Flooding**
* **Content Addressable Memory (CAM) Table:** Hardware memory array on Layer 2 switches mapping destination MAC addresses to physical switch ports with associated aging timers.
* **MAC Flooding Mechanism:** Tools (e.g., `macof`) generate a massive volume of invalid source MAC and IP addresses (thousands per second) to saturate the CAM table's finite memory buffer.
* **Fail-Open Mode:** When the CAM table reaches full capacity, the switch cannot learn new MAC addresses and drops back to broadcasting incoming unicast frames to **all** switch ports (behaving like a hub), allowing passive interception of third-party traffic.

**Switch Port Stealing**
Exploits switch learning behavior by racing to bind a target's MAC address to the attacker's switch port.
* **Mechanism:** The attacker floods the switch with forged ARP source frames containing the target's MAC address at high frequency. The switch rapidly updates its CAM table, steering legitimate return packets meant for the target host directly to the attacker's port.

---

### 3. Layer 3 & Protocol Exploitation (ARP & DHCP)

```
                       [ ARP POISONING / MITM FLOW ]
                       
  [ Victim Host ]  ◄─── "192.168.1.1 is at ATTACKER_MAC" ─── [ Attacker (Cain/Ettercap) ]
 (192.168.1.50)                                                    │
        │                                                          │
        └─────── "192.168.1.50 is at ATTACKER_MAC" ────────────────┘
                                   │
                                   ▼
                            [ Gateway Router ]
                              (192.168.1.1)
```

**ARP Poisoning / ARP Spoofing**
Exploits the stateless nature of the Address Resolution Protocol (ARP), which accepts unsolicited ARP replies (`Opcode 2`) without cross-referencing prior requests (`Opcode 1`).
* **Poisoning Mechanism:** The attacker sends gratuitous, unsolicited ARP responses to the Target Host and Gateway Router simultaneously, overwriting their dynamic ARP caches:
  * Overwrites Gateway IP ($192.168.1.1$) entry in Target Cache with $ATTACKER\_MAC$.
  * Overwrites Target IP ($192.168.1.50$) entry in Gateway Cache with $ATTACKER\_MAC$.
* **Man-In-The-Middle (MITM):** Enables full full-duplex traffic interception, content inspection, and transparent forwarding using IP forwarding (`sysctl -w net.ipv4.ip_forward=1`).

**DHCP Starvation & Rogue DHCP Attacks**
* **DHCP Starvation:** Attackers generate thousands of forged `DHCPDISCOVER` packets with randomized client MAC addresses (using `Yersinia` or `piggy`).
  * **Result:** Exhausts the DHCP server's available IP address pool (Scope depletion), preventing legitimate network hosts from acquiring valid network configurations.
* **Rogue DHCP Server Attack:** Executed in tandem with DHCP Starvation. Once the legitimate DHCP server is exhausted, the attacker launches a rogue DHCP server to respond to new `DHCPDISCOVER` requests, assigning victims:
  * Attacker's IP as Default Gateway (enabling Layer 3 MITM).
  * Attacker's IP as Primary DNS Server (enabling DNS Spoofing).

---

### 4. IRDP & DNS Spoofing

**ICMP Router Discovery Protocol (IRDP) Spoofing**
* Attackers broadcast periodic unsolicited IRDP Router Advertisement messages (`ICMP Type 9, Code 0`) configured with high preference values.
* Target hosts automatically adjust their local routing tables to prefer the attacker's host as the default gateway for outbound internet traffic.

**DNS Spoofing / DNS Cache Poisoning**
* **Intranet DNS Spoofing:** Executed via ARP Poisoning. Intercepts local port 53 DNS queries and injects forged DNS response packets containing malicious IP mappings before the real server responds.
* **Internet DNS Poisoning (Kaminsky Attack):** Injects malicious resource records (`A` records or `NS` authority delegations) directly into recursive DNS server caches using transaction ID guessing.

---

### 5. Defensive Technologies & Mitigations

```
                      [ LAYER 2 DEFENSE ARCHITECTURE ]
                                      │
            ┌─────────────────────────┼─────────────────────────┐
            ▼                         ▼                         ▼
   [ PORT SECURITY ]           [ DHCP SNOOPING ]     [ DYNAMIC ARP INSPECTION ]
- Enforces MAC Limits       - Drops Unauthorized       - Validates ARP Packets
- Drops Spoofed Frames        DHCP Requests             against DHCP Binding Table
- Disables Switch Port        - Categorizes Ports as     - Blocks Unsolicited ARP
  (Err-Disable Mode)          Trusted / Untrusted       Replies (Poisoning Mitigation)
```

* **Port Security:** Restricts input to a switch port by limiting the maximum number of dynamic MAC addresses learned. Configures violation actions:
  * `Protect`: Drops frames with unknown source MACs; no alert generated.
  * `Restrict`: Drops frames with unknown source MACs; logs syslog alert and increments violation counter.
  * `Shutdown`: Immediately shuts down the port (`err-disable` state) upon violation.
* **DHCP Snooping:** Configures switch ports into **Trusted** (connected to legitimate DHCP servers) and **Untrusted** (connected to end-user systems) interfaces. Untrusted ports are blocked from sourcing `DHCPOFFER` or `DHCPACK` packets. Build and maintain the **DHCP Snooping Binding Database** (`MAC-to-IP-to-VLAN-to-Port`).
* **Dynamic ARP Inspection (DAI):** Intercepts, logs, and discards invalid ARP requests and responses on untrusted ports by cross-referencing packet contents against the **DHCP Snooping Binding Database**.

---

## Tier 2: Tool & CLI Mechanics

---

### 1. Active Sniffing Tools & Commands

**macof (Dsniff Suite)**
Performs high-speed MAC flooding by generating random MAC and IP address pairs to overflow switch CAM tables.
```bash
# Basic MAC flooding on default interface
macof -i eth0 -n 100000
```
* **Flags:**
  * `-i`: Specifies network interface.
  * `-s`: Sets explicit source IP address.
  * `-d`: Sets explicit destination IP address.
  * `-n`: Specifies total number of packets to transmit.

**Yersinia**
Multi-protocol Layer 2 attack tool targeting infrastructure protocols including DHCP, STP, CDP, DTP, 802.1q, and HSRP.
```bash
# Launch Yersinia in interactive Ncurses GUI mode
yersinia -I

# Launch automated DHCP starvation attack via CLI
yersinia dhcp -m starvation -i eth0
```

**Cain & Abel**
Windows-based password recovery and active sniffing platform.
* Executes ARP Poisoning (Routing section), MAC Spoofing, Promiscuous Mode detection, and credentials extraction (HTTP, FTP, POP3, RDP, SMB).

---

### 2. Network Analysis & Packet Inspection (Wireshark & CLI)

**Wireshark Display Filters**
Used during packet capture analysis to isolate malicious indicators, plaintext credentials, and protocol anomalies:

```wireshark
# Isolate HTTP POST requests (Forms & Plaintext Login Submissions)
http.request.method == "POST"

# Isolate Plaintext Authentication across Protocols
http.authorization || ftp.request.command == "PASS" || pop3

# Detect Duplicate IP warnings (ARP Poisoning Indicator)
arp.duplicate-address-detected || arp.opcode == 2

# Isolate DHCP Traffic (Discovery, Offer, Request, Ack)
bootp || dhcp

# Filter ICMP Router Advertisements (IRDP Attacks)
icmp.type == 9
```

**Tcpdump CLI Usage**
Lightweight packet analysis tool for command-line environments.
```bash
# Capture raw HTTP traffic containing POST requests on eth0
tcpdump -i eth0 -nn -s 0 -A 'tcp port 80 and (tcp[13] & 8 != 0)'

# Capture ARP packets to file for offline inspection
tcpdump -i eth0 -w arp_capture.pcap arp
```

---

### 3. Sniffer Detection Mechanics

```
                  [ PROMISCUOUS MODE ARP DETECTION ]
                  
  [ Scanner / Nmap ] ──( ARP Request to Fake MAC: FF:FF:FF:FF:FF:FE )──► [ Target NIC ]
                                                                               │
       ┌───────────────────────────────────────────────────────────────────────┴───────────────────┐
       ▼                                                                                           ▼
 [ Normal Mode NIC ]                                                                    [ Promiscuous Mode NIC ]
- Drops frame at hardware layer due to MAC mismatch.                                   - Passes frame to OS kernel.
- NO ARP Response returned.                                                            - Responds with valid ARP Reply!
```

**Nmap Promiscuous Mode Detection (NSE)**
Uses non-standard or fake broadcast destination MAC addresses (`FF:FF:FF:FF:FF:FE`) in ARP requests to identify sniffing nodes. Standard NIC drivers discard these packets, but NICs operating in promiscuous mode accept and reply to them.
```bash
# Execute Nmap script to detect network sniffers on local subnet
nmap --script=sniffer-detect 192.168.1.0/24
```

**Promiscuous Detection Software Tools:**
* **PromiScan / PromQry:** Windows GUI utilities designed to detect remote network interfaces running in promiscuous mode across local subnets.
* **Ifconfig / Ip link (Linux Local Checks):**
  ```bash
  # Check local network interface flags for PROMISC status
  ip link show eth0 | grep -i PROMISC
  ```

---

## Tier 3: Attack Chains & Execution Scenarios

---

### Scenario 1: Full Layer 2 MITM & Password Harvesting Chain

```
                          [ ATTACK CHAIN: MITM & HARVESTING ]
                          
[ Host Discovery ]      [ ARP Poisoning ]       [ IP Forwarding ]     [ Traffic Sniffing ]   [ Credential Extraction ]
 Scan Subnet Subnet  ➔ Transmit Unsolicited  ➔ Enable Kernel   ➔ Capture Ingress   ➔ Apply Filter for
 via Cain/Nmap          ARP Replies (Opcode 2) IP Forwarding      Unencrypted Traffic    HTTP POST / FTP PASS
```

1. **Host Discovery:** Attacker executes an ARP sweep using Nmap or Cain & Abel to map active IP-to-MAC bindings on subnet `192.168.1.0/24`.
2. **ARP Poisoning Execution:** Attacker sends continuous, unsolicited ARP reply packets (`Opcode 2`) to Victim ($192.168.1.50$) and Gateway ($192.168.1.1$), binding both IP addresses to $ATTACKER\_MAC$.
3. **IP Forwarding Enabling:** Attacker turns on kernel packet forwarding to allow intercepted traffic to flow transparently without dropping packets:
   ```bash
   sysctl -w net.ipv4.ip_forward=1
   ```
4. **Packet Capture & Analysis:** Attacker opens Wireshark on the active interface (`eth0`) and applies the filter `http.request.method == "POST"`.
5. **Credential Extraction:** Victim logs into an unencrypted HTTP administrative panel. Wireshark captures the frame and exposes the `username` and `password` variables transmitted in HTML form data.

---

### Scenario 2: DHCP Starvation & Rogue Server Redirect Chain

1. **Target Identification:** Attacker connects to an enterprise switch port and identifies the legitimate DHCP server address ($10.0.0.1$).
2. **Exhaustion Attack:** Attacker launches `yersinia dhcp -m starvation -i eth0`, continuously broadcasting thousands of `DHCPDISCOVER` packets with randomized source MAC addresses.
3. **Resource Starvation:** The legitimate DHCP server exhausts its entire scope ($10.0.0.100 - 10.0.0.254$) and stops responding to new IP requests.
4. **Rogue Server Activation:** Attacker initializes a rogue DHCP server on their local machine, configuring it to distribute:
   * Dynamic IP Pool: $10.0.0.50 - 10.0.0.99$
   * Default Gateway: $10.0.0.15$ (Attacker Host)
   * Primary DNS Server: $10.0.0.15$ (Attacker Host)
5. **Victim Interception:** New workstations joining the network accept configuration parameters from the rogue server, redirecting all external web and DNS traffic through the attacker's system.

---

## Tier 4: CEH v13 Exam Triggers

---

### Key Trigger Phrases & Exam Rules

| Exam Question Phrase | Correct Answer / Concept |
| :--- | :--- |
| *"Floods switch with thousands of random MAC addresses causing it to enter fail-open mode"* | **MAC Flooding (`macof`)** |
| *"Switch drops back to broadcasting all frames to all ports like a hub when full"* | **Fail-Open / CAM Table Saturation** |
| *"Unsolicited ARP replies sent to overwrite dynamic ARP tables of target and gateway"* | **ARP Poisoning / ARP Spoofing** |
| *"Exhausts DHCP server scope by sending continuous DHCPDISCOVER packets with fake MACs"* | **DHCP Starvation Attack (`Yersinia`)** |
| *"Rogue server configured to hand out attacker's IP as Default Gateway and DNS"* | **Rogue DHCP Server Attack** |
| *"Sends ARP requests with a non-standard broadcast destination MAC address (FF:FF:FF:FF:FF:FE)"* | **`nmap --script=sniffer-detect`** |
| *"Switch feature that validates incoming ARP responses against a trusted binding database"* | **Dynamic ARP Inspection (DAI)** |
| *"Security feature that restricts switch ports from responding to rogue DHCP requests"* | **DHCP Snooping (Trusted vs Untrusted Ports)** |
| *"Port security violation mode that drops frames and shuts down the interface entirely"* | **`Shutdown` (Err-Disable Mode)** |
| *"Wireshark display filter used to isolate form submission passwords in HTTP traffic"* | **`http.request.method == "POST"`** |
| *"Wireshark display filter used to identify potential ARP Poisoning attacks"* | **`arp.duplicate-address-detected`** |
| *"Port mirroring technique used on managed switches to forward copy of traffic to a sniffer"* | **SPAN Port (Switched Port Analyzer)** |
| *"NIC setting that overrides hardware destination address filtering to process all captured frames"* | **Promiscuous Mode** |
| *"Uses ICMP Type 9 router advertisements with high preference values to alter default routing"* | **IRDP Spoofing** |