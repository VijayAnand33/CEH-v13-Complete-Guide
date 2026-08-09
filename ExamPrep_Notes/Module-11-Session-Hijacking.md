# Module 11: Session Hijacking

## Tier 1: Core Concepts & Principles

### Session Hijacking Fundamentals
* **Session Hijacking Definition:** The exploitation of a valid computer session (session key, token, or cookie) to gain unauthorized access to information or services in a computer system without re-authenticating.
* **Session Hijacking vs. Session Fixation vs. Spoofing:**
  * **Session Hijacking:** Stealing or intercepting an active, valid session token *after* the legitimate user has already authenticated.
  * **Session Fixation:** Injecting or forcing a known, valid session token onto a user's browser *before* they authenticate, then taking over the session after they log in.
  * **Spoofing:** Impersonating a system or user address (IP, MAC) before or during connection establishment, without necessarily relying on an existing authenticated application token.
* **Attack Classifications:**
  * **Passive Hijacking:** Monitoring network traffic for unencrypted session IDs, cookies, or credentials (sniffing/eavesdropping) without altering packet data.
  * **Active Hijacking:** Intercepting, injecting, modifying, or dropping packets in transit (e.g., Man-in-the-Middle, TCP sequence prediction, session token forging) to take over an established connection.
* **Session Lifecycle Vectors:** Attackers exploit flaws in session generation (predictable tokens), transmission (cleartext HTTP), storage (insecure client-side cookies), or expiration (infinite session timeouts).

---

## Tier 2: Technical Analysis & Mechanics

### Session Hijacking Attack Categories & Vectors

| Category | Attack Vector | Technical Execution Mechanism | Primary Impact |
| :--- | :--- | :--- | :--- |
| **Network-Level** | **TCP Sequence Prediction** | Predicting TCP sequence numbers (32-bit `SEQ` field) to inject spoofed packets into an active stream. | TCP Connection Hijacking |
| **Network-Level** | **ARP Poisoning / MITM** | Gratuitous ARP responses map attacker's MAC address to gateway/victim IP addresses. | Complete traffic interception and injection |
| **Network-Level** | **IP Spoofing / Source Routing** | Forging source IP header fields or enforcing strict source routing paths to bypass ACLs. | Traffic redirection and response sniffing |
| **Application-Level** | **Cookie Theft / XSS** | Injecting malicious JavaScript (`document.cookie`) to exfiltrate session cookies to C2 servers. | Direct user session takeover |
| **Application-Level** | **Session Side-Jacking** | Sniffing unencrypted HTTP cookie strings over unencrypted Wi-Fi networks (HTTP traffic). | Web session impersonation |
| **Application-Level** | **Session Token Prediction** | Brute-forcing or calculating sequential/weakly-hashed session identifiers (e.g., timestamps). | Automated session brute-forcing |

### Network-Level TCP Hijacking Mechanics
* **TCP Handshake Review:** Synchronization (`SYN`) $\rightarrow$ Sync-Acknowledge (`SYN-ACK`) $\rightarrow$ Acknowledge (`ACK`).
* **Sequence & Acknowledgment Numbers:**
  * Each byte of data transmitted via TCP is tracked using a 32-bit Sequence Number (`SEQ`).
  * The receiving host responds with an Acknowledgment Number (`ACK`) indicating the next expected byte.
* **Desynchronization Process:**
  1. Attacker monitors active TCP connection between Victim and Server.
  2. Attacker sends an `RST` or `FIN` packet to the victim, or floods the victim (DoS) to desynchronize its TCP state.
  3. Attacker spoofs the victim's IP address and injects crafted TCP packets to the server with the correctly predicted next `SEQ` / `ACK` values.
  4. Server accepts injected data as authentic, granting the attacker interactive access.

---

## Tier 3: CLI, Tools & Framework Syntax

### Interception Proxies & Traffic Analysis Tools
* **Caido / Hetty / Burp Suite:** Interception proxies used to capture, queue, modify, and repeat HTTP/HTTPS traffic.
* **Bettercap:** Modern network attack framework used for ARP poisoning, DNS spoofing, and credentials/cookie sniffing.
* **Wireshark:** Network protocol analyzer used to capture live packet streams and inspect ARP, TCP, and HTTP protocols.

### Key Commands & Syntax Execution

```bash
# Bettercap: Launch interactive session and activate network discovery/sniffing
bettercap -iface eth0

# Bettercap Internal Commands: Enable discovery and packet interception
net.probe on
net.recon on
net.sniff on

# Bettercap ARP Spoofing Module Setup
set arp.spoof.targets <Target_Victim_IP>
set arp.spoof.fullduplex true
arp.spoof on

# Wireshark Display Filters for Session Hijacking Detection
# Filter for ARP traffic to identify ARP poisoning / duplicate MACs
arp

# Filter for specific HTTP POST requests containing sensitive parameters
http.request.method == "POST"

# Filter for session cookie strings in HTTP headers
http.cookie contains "PHPSESSID"
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Defensive Mitigations & Session Security Standards
* **Transport Layer Security (TLS/HTTPS):** Enforce end-to-end TLS encryption across all endpoints. Implement **HSTS (HTTP Strict Transport Security)** to prevent SSL Stripping attacks.
* **Secure Cookie Attributes:**
  * `Secure` Flag: Guarantees cookies are transmitted exclusively over encrypted HTTPS connections.
  * `HttpOnly` Flag: Restricts client-side scripts (JavaScript) from accessing `document.cookie`, preventing XSS-based cookie theft.
  * `SameSite` Flag: Restricts cookie transmission in cross-site requests (`SameSite=Strict` or `SameSite=Lax`) to mitigate Cross-Site Request Forgery (CSRF).
* **Session Lifecycle Hardening:**
  * Regenerate session tokens immediately following user authentication (prevents Session Fixation).
  * Enforce strict idle session timeouts and absolute session termination windows.
  * Bind session tokens to secondary client parameters (e.g., User-Agent, client IP subnet hashing).
* **Network-Level Hardening:**
  * Deploy **Dynamic ARP Inspection (DAI)** on network switches to validate ARP requests against a trusted DHCP Snooping binding database.
  * Implement **DHCP Snooping** and IEEE **802.1X** network access control.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **Session Hijacking** | Stealing an active token *after* successful login | Targets established, authenticated sessions |
| **Session Fixation** | Forcing a known session token *before* user logs in | Attacker provides the token; victim authenticates it |
| **Passive Hijacking** | Eavesdropping / sniffing cleartext HTTP tokens | Does not modify network traffic or inject packets |
| **Active Hijacking** | Intercepting, injecting, or altering active packets | Modifies packet streams, uses MITM or TCP prediction |
| **Desynchronization** | Forcing victim offline or altering TCP `SEQ` counts | Essential step in network-level TCP connection takeover |
| **`HttpOnly` Cookie Flag** | Prevents client-side JavaScript access (`document.cookie`) | Primary defense against Cross-Site Scripting (XSS) cookie theft |
| **`Secure` Cookie Flag** | Forces browser to transmit cookie *only* over HTTPS | Prevents plain-text sniffing over unencrypted HTTP |
| **Dynamic ARP Inspection (DAI)** | Switch-level defense against ARP poisoning / spoofing | Validates ARP packets against DHCP Snooping database |
| **HSTS** | Forces browser to use HTTPS exclusively | Prevents SSL Stripping / downgrade attacks |
| **Bettercap `net.sniff`** | Live packet sniffing and network reconnaissance module | Generates ARP probes and captures traffic in MITM mode |