# Module 10 — Denial-of-Service (DoS/DDoS)

## Tier 0: Low-Level Network & Protocol Fundamentals

### TCP Handshake & Connection State Dynamics
```text
  Client (Attacker / Zombie / Victim)                 Server (Target)
              |                                             |
              | -------------- SYN (Seq=X) --------------> |  State: SYN-RCVD
              |                                             |  Allocates TCB Memory Buffer
              | <------- SYN-ACK (Seq=Y, Ack=X+1) --------- |  Awaits final ACK
              |                                             |
              | -------------- ACK (Ack=Y+1) -------------> |  State: ESTABLISHED
```
* **Transmission Control Block (TCB):** A kernel-level memory structure allocated for every incoming `SYN` request to keep track of connection metadata, sequence numbers, and window states.
* **SYN Backlog Queue:** A fixed-capacity memory buffer where connections in the `SYN-RCVD` state sit waiting for the client's final `ACK`. Filling this queue starves legitimate users of new connection slots.
* **Stateful vs. Stateless Vulnerabilities:**
  * **TCP (OSI Layer 4):** Connection-oriented and stateful. Targetable via resource/state-exhaustion vectors.
  * **UDP (OSI Layer 4):** Connectionless and stateless. Targetable via volumetric floods, packet-rate overload, and unverified IP-spoofed reflection vectors.
  * **ICMP (OSI Layer 3):** Diagnostic and operational protocol (`Echo Request` Type 8, `Echo Reply` Type 0). Exploited via broadcast reflections, malformed sizing, and link saturation.

---

## Tier 1: Theoretical Architecture & DDoS Classifications

### Definitions & Infrastructure
* **Denial-of-Service (DoS):** An attack from a single source designed to consume target resources, exhaust bandwidth, or crash a system to deny service to legitimate users.
* **Distributed Denial-of-Service (DDoS):** A coordinated attack using multiple geographically distributed compromised hosts (**Bots / Zombies**) managed via a **Command and Control (C2) / Bot Herder** network to flood a single target concurrently.

### The 3 Structural Vectors of DDoS
1. **Volumetric Attacks:** Aimed at consuming network pipeline capacity or interface processing limits.
   * *Metrics:* Bits per Second (Bps), Packets per Second (Pps).
2. **Protocol / State-Exhaustion Attacks:** Aimed at exhausting finite connection state tables in intermediate appliances (firewalls, load balancers) or kernel memory structures (TCB queues).
   * *Metrics:* Connections per Second (Cps), Concurrent Connection Limits.
3. **Application Layer (Layer 7) Attacks:** Aimed at overloading backend web server application logic, CPU routines, database transaction pools, or RAM.
   * *Metrics:* Requests per Second (Rps).

---

## Tier 2: Attack Mechanics & Execution Vectors

### 1. Volumetric & Protocol Attack Vectors
* **SYN Flooding Attack:**
  * **Mechanics:** Attacker sends a mass volume of `SYN` packets with spoofed/randomized source IP addresses. Target issues `SYN-ACK` responses and allocates TCB memory entries in its backlog queue. Target holds state until timeouts expire, saturating the backlog and dropping legitimate `SYN` attempts.
* **Smurf Attack:**
  * **Mechanics:** ICMP-based broadcast reflection attack. Attacker sends ICMP `Echo Requests` to an intermediary network's **broadcast address**, spoofing the **victim's IP as the source**. Every active host on the intermediary network sends ICMP `Echo Replies` back to the victim, saturating their link.
* **Fraggle Attack:**
  * **Mechanics:** UDP-based variant of Smurf. Attacker sends spoofed UDP packets targeting UDP port 7 (Echo) or port 19 (Chargen) to an intermediary broadcast address, causing hosts to flood the victim with UDP traffic.
* **Ping of Death (PoD):**
  * **Mechanics:** Attacker constructs oversized or corrupted ICMP/IP packets exceeding the maximum IPv4 packet limit of **65,535 bytes**. Reassembly of these oversized IP fragments causes a buffer overflow and crashes the target OS kernel.
* **Teardrop Attack:**
  * **Mechanics:** IP fragmentation attack using overlapping fragment offset values (`Fragment Offset` + `Total Length` overlap). When the victim OS attempts to reassemble the fragments, memory allocation logic breaks, causing kernel panics or crashes.
* **LAND Attack (Local Area Network Denial):**
  * **Mechanics:** Attacker crafts a TCP `SYN` packet where the **Source IP and Source Port are forged to be identical to the Target IP and Target Port**. The target machine enters a recursive feedback loop trying to reply to itself, resulting in state exhaustion or crash.
* **Peer-to-Peer (P2P) DDoS Attacks:**
  * **Mechanics:** Directs thousands of clients on P2P file-sharing networks (e.g., BitTorrent) to request shared files or data blocks directly from the target's IP address.

### 2. Amplification & Reflection DDoS Vectors
* **Reflection Architecture:** Sending queries to open public servers (DNS, NTP, SNMP, Memcached) while forging the source IP to match the target's IP address, causing responses to be directed at the victim.
* **Amplification Factor:** The ratio comparing the size of the request payload sent by the attacker to the size of the response payload generated by the reflector server toward the victim.
  * **DNS Amplification:** Uses `EDNS0` extensions requesting `ANY` records from open recursive DNS resolvers. *Amplification Ratio:* **20x to 50x**.
  * **NTP Amplification:** Sends `monlist` or `req_mon` diagnostic commands to open NTP servers, returning the IP addresses of the last 600 hosts that synced time with that server. *Amplification Ratio:* Up to **550x**.
  * **Memcached Amplification:** Sends UDP queries to misconfigured Memcached databases (UDP port 11211). *Amplification Ratio:* **10,000x to 50,000x**.
  * **SSDP / UPnP Amplification:** Sends Simple Service Discovery Protocol requests on UDP port 1900 to expose connected devices.

### 3. Application Layer (Layer 7) Vectors
* **HTTP Flood:** Generates high volumes of valid-looking HTTP `GET` or `POST` requests to resource-heavy endpoints (e.g., complex search forms, dynamic database calls, document/PDF renders).
* **Slowloris:**
  * **Mechanics:** Opens multiple concurrent HTTP connections to a web server (e.g., Apache) and sends **partial/incomplete HTTP headers** at slow, periodic intervals (e.g., `X-a: b\r\n` every 10–15 seconds). By withholding the terminating double CRLF sequence (`\r\n\r\n`), the server keeps thread pools occupied waiting for the headers to finish, exhausting connection slots.
* **R-U-Dead-Yet (RUDY):**
  * **Mechanics:** Slow-rate HTTP `POST` attack targeting web form fields. Declares a large `Content-Length` header value, but sends the form payload body at a rate of 1 byte every few seconds, starving server thread pools.
* **Multi-Vector DDoS:** Attack campaigns that combine volumetric bandwidth flooding, protocol state exhaustion, and HTTP application attacks simultaneously to overwhelm defenses across multiple layers.

---

## Tier 3: Tool Matrix & Command-Line Execution

### Tool Matrix
| Tool Name | Primary Vector / Category | Technical Features / Capabilities |
| :--- | :--- | :--- |
| **LOIC (Low Orbit Ion Cannon)** | Volumetric DoS | GUI utility flooding TCP, UDP, or HTTP. Features an unencrypted IRC-driven "Hivemind" remote control mode; reveals the true IP if unproxied. |
| **HOIC (High Orbit Ion Cannon)** | Volumetric DDoS | Multi-threaded HTTP `GET`/`POST` flooding tool. Uses customizable **Booster Scripts (`.hoic`)** to randomize headers and bypass signature-based blocking. |
| **Slowloris** | Layer 7 DoS | Script designed to exhaust HTTP thread pools by maintaining open connections via incomplete HTTP headers. |
| **RUDY (R-U-Dead-Yet)** | Layer 7 DoS | Form submission starvation tool executing slow-rate HTTP `POST` requests with large `Content-Length` headers. |
| **Hping3** | CLI Packet Crafting | Multi-purpose CLI packet generator capable of crafting raw TCP, UDP, ICMP, and custom-flagged packet floods. |
| **Anti DDoS Guardian** | Defensive & Analysis | Windows host-based security software used to monitor active socket connections, detect incoming packet floods, and block offending IPs automatically. |

### Critical Hping3 Command Executions

#### 1. Standard TCP SYN Flood
```bash
hping3 -S --flood -V -p 80 10.10.10.50
```
* `-S`: Enables TCP SYN flag.
* `--flood`: Sends packets as fast as possible (suppresses reply output).
* `-V`: Enables verbose output.
* `-p 80`: Target port 80.

#### 2. TCP SYN Flood with Explicit Source IP Spoofing
```bash
hping3 -S -a 192.168.1.254 -p 80 --flood 10.10.10.50
```
* `-a 192.168.1.254`: Forces the source IP of all generated packets to `192.168.1.254`.

#### 3. TCP SYN Flood with Fully Randomized Source IPs
```bash
hping3 -S --rand-source -p 80 --flood 10.10.10.50
```
* `--rand-source`: Generates a pseudo-random spoofed source IP for every outgoing packet to bypass IP-based rate limiting.

#### 4. UDP Flood
```bash
hping3 --udp --flood -p 53 10.10.10.50
```
* `--udp`: Switches mode from default TCP to UDP datagrams.

#### 5. ICMP Flood
```bash
hping3 --icmp --flood 10.10.10.50
```
* `--icmp`: Sends rapid ICMP `Echo Request` packets.

#### 6. LAND Attack Simulation
```bash
hping3 -S -a 10.10.10.50 -k -s 80 -p 80 --flood 10.10.10.50
```
* `-a 10.10.10.50`: Sets source IP identical to target IP.
* `-s 80 -p 80`: Sets source port and destination port to port 80.

---

## Tier 4: Defense, Mitigation & Countermeasure Matrix

### Mitigation Controls
* **SYN Cookies:**
  * **Mechanics:** Avoids TCB queue allocation upon receiving a `SYN`. Encodes initial connection state into the Initial Sequence Number (`ISN`) returned in the `SYN-ACK`. TCB allocation occurs **only after** receiving a valid final `ACK` containing the hashed sequence value.
* **Source Address Validation (BCP 38 / RFC 2827):** Ingress/egress packet filtering deployed at the ISP router level to verify that outgoing packets match authorized network prefixes, blocking spoofed source IP packets.
* **BGP Anycast Routing:** Routes traffic to the nearest topological data center within a globally distributed network sharing identical IP addresses, dispersing volumetric attack loads across scrubbing nodes.
* **Scrubbing Centers / Reverse Cloud Proxies:** In-line inspection infrastructure (e.g., Cloudflare, Akamai, AWS Shield) that intercepts traffic, strips malicious signatures, validates clients via JavaScript challenges or CAPTCHAs, and forwards clean traffic to origin servers.
* **Blackhole Routing / Null Routing:** Routes traffic targeting a compromised IP address to a non-existent virtual interface (`/dev/null`) at the ISP level, dropping all incoming traffic to protect core routing links.
* **Sinkholing:** Modifies DNS resolution or network routing to steer attack traffic toward an isolation network for traffic analysis and packet inspection.
* **Rate Limiting & Traffic Shaping:** Enforces policy caps on connection rates, packet rates, or bandwidth usage per source IP per unit of time.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

### Scenario & Concept Triggers
| Question Stem Keyword / Scenario | Correct Answer / Concept |
| :--- | :--- |
| Unanswered `SYN` packets fill memory backlog queue | **SYN Flood** |
| ICMP `Echo Request` directed to **broadcast address** with **spoofed target IP** | **Smurf Attack** |
| UDP requests directed to **broadcast address** with **spoofed target IP** | **Fraggle Attack** |
| Packet sizing exceeds maximum IPv4 limit of **65,535 bytes** | **Ping of Death (PoD)** |
| Overlapping IP fragment offsets cause OS crash during reassembly | **Teardrop Attack** |
| **Source IP and Source Port are identical** to Target IP and Target Port | **LAND Attack** |
| Keeps web server HTTP connections open using **incomplete headers** (`\r\n`) | **Slowloris** |
| Slow HTTP `POST` submission declaring large `Content-Length` | **R-U-Dead-Yet (RUDY)** |
| Exploits `monlist` query on open servers (~550x amplification factor) | **NTP Amplification** |
| Exploits UDP port 11211 yielding massive (~10,000x+) amplification | **Memcached Amplification** |
| Defensive mechanism that **encodes connection state in `ISN`** to prevent memory allocation | **SYN Cookies** |
| ISP-level network standard that **prevents IP spoofing** at the router level | **BCP 38 / RFC 2827** |
| ISP mitigation routing attack traffic directly to `/dev/null` | **Blackhole / Null Routing** |

### Command Syntax & Tool Triggers
| Command / Feature Trigger | Exam Target |
| :--- | :--- |
| `hping3 -S --flood --rand-source -p 80` | **SYN Flood with Randomized Source IPs** |
| `hping3 -S -a [IP] -p 80 --flood` | **SYN Flood with Explicit Spoofed Source IP** |
| Tool using customizable **Booster Scripts (`.hoic`)** to bypass signatures | **HOIC** |
| Legacy GUI tool with an IRC-based "Hivemind" mode | **LOIC** |
| Windows host security tool used to monitor connections and auto-block flooding IPs | **Anti DDoS Guardian** |