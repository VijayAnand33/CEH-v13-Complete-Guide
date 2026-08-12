# Module 16: Hacking Wireless Networks

## Tier 1: Core Concepts & Principles

### IEEE 802.11 Wireless Architecture & Frame Taxonomy
* **IEEE 802.11 Standard:** The international physical (PHY) and data link (MAC) layer standard governing Wireless Local Area Networks (WLANs).
* **Wireless Network Entities:**
  * **SSID (Service Set Identifier):** Human-readable network identifier name broadcast in Beacon frames (up to 32 bytes).
  * **BSSID (Basic Service Set Identifier):** The unique 48-bit MAC address of the Access Point's (AP) wireless radio.
  * **ESSID (Extended Service Set Identifier):** Identifies a collection of interconnected BSSIDs/APs operating under a single extended network domain.
* **Operating Modes:**
  * **Managed Mode (Station Mode):** Default mode where client NIC connects to a specific Access Point.
  * **Monitor Mode (RFMON):** Unlinks the wireless card from any BSSID, allowing the interface to passively capture all raw 802.11 frames transmitted on a chosen radio channel without associating.
* **802.11 Frame Categories:**
  1. **Management Frames:** Network discovery and association (`Beacon`, `Probe Request/Response`, `Authentication`, `Deauthentication`, `Disassociation`).
  2. **Control Frames:** Transmission coordination and channel reservation (`RTS` - Request to Send, `CTS` - Clear to Send, `ACK` - Acknowledgment).
  3. **Data Frames:** Transmits actual higher-layer TCP/IP payloads across the wireless medium.

---

## Tier 2: Technical Analysis & Mechanics

### WPA/WPA2 4-Way Handshake & Cracking Mechanics
* **WPA2-PSK (Pre-Shared Key) Architecture:** Employs AES-CCMP (Counter Mode with Cipher Block Chaining Message Authentication Code Protocol) for frame encryption. Authentication relies on a 256-bit Pairwise Master Key (PMK).
* **PMK Calculation Formula:**
  $$\text{PMK} = \text{PBKDF2}(\text{Passphrase}, \text{SSID}, \text{SSID Length}, 4096, 256)$$
* **4-Way Handshake Workflow:**
  1. **Message 1 (AP $\rightarrow$ Client):** AP sends `ANonce` (Authenticator Nonce) to Client.
  2. **Message 2 (Client $\rightarrow$ AP):** Client computes `PTK` (Pairwise Transient Key) using `ANonce` + `SNonce` (Supplicant Nonce) + `AP MAC` + `Client MAC` + `PMK`. Sends `SNonce` and a Message Integrity Code (`MIC`) to AP.
  3. **Message 3 (AP $\rightarrow$ Client):** AP verifies `MIC`, generates `GTK` (Group Temporal Key), and sends `GTK` + `MIC` to Client.
  4. **Message 4 (Client $\rightarrow$ AP):** Client sends final `ACK` confirming key installation.
* **WPA2 Offline Cracking Mechanics:**
  * Attackers capture a valid 4-Way Handshake file (`.cap` / `.pcap`).
  * Aircrack-ng takes candidate wordlist passwords, computes candidate PMKs using the network SSID, derives the candidate PTK using captured Nonces and MAC addresses, and calculates a candidate `MIC`.
  * If the candidate `MIC` matches the captured Message 2 or Message 3 `MIC`, the passphrase is successfully recovered.

### WPA3 Security Enhancements (SAE)
* **Simultaneous Authentication of Equals (SAE):** Replaces the WPA2 pre-shared key 4-way handshake with a Dragonfly Key Exchange (Diffie-Hellman variant).
* **Defensive Advantage:** Neutralizes offline dictionary attacks completely (forward secrecy) and renders passive handshake captures immune to offline cracking.

---

## Tier 3: CLI, Tools & Framework Syntax

### Core Wireless Assessment Tooling
* **Aircrack-ng Suite:** Complete framework for wireless monitoring, deauthentication, handshake capturing, and offline key recovery.
  * `airmon-ng`: Interface monitor mode management script.
  * `airodump-ng`: Raw 802.11 packet capture engine.
  * `aireplay-ng`: Packet injection tool used to generate traffic, deauthenticate clients, and force 4-way handshakes.
  * `aircrack-ng`: 802.11 key cracking utility (WEP / WPA / WPA2-PSK).
* **Wireshark:** Network packet analyzer with built-in Radiotap header parsing and 802.11 display filter support.

### Key Commands & Execution Syntax

```bash
# Enable Monitor Mode on Wireless Interface (airmon-ng)
sudo airmon-ng start wlan0

# Verify Monitor Mode State
iwconfig

# Capture All Wireless Networks / BSSIDs in Range (airodump-ng)
sudo airodump-ng wlan0mon

# Target Specific BSSID and Channel to Capture 4-Way Handshake
sudo airodump-ng --bssid <AP_MAC> -c <CHANNEL> -w /tmp/WPA2capture wlan0mon

# Inject Deauthentication Frames to Force Client Re-Authentication & Capture Handshake (aireplay-ng)
sudo aireplay-ng --deauth 10 -a <AP_MAC> -c <VICTIM_CLIENT_MAC> wlan0mon

# Perform Offline WPA2 Dictionary Attack against Handshake File (aircrack-ng)
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b <AP_MAC> /tmp/WPA2capture-01.cap

# Wireshark Display Filters for 802.11 Traffic Analysis
# Filter for Beacon Management Frames
wlan.fc.type_subtype == 0x0008

# Filter for WPA/WPA2 EAPOL 4-Way Handshake Packets
eapol

# Filter for Deauthentication Frames
wlan.fc.type_subtype == 0x000c
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Wireless Network Hardening & Defense Architecture
* **Upgrade Encryption Standards:** Transition legacy WEP, WPA, and WPA2-PSK networks to **WPA3-Enterprise** or **WPA3-Personal (SAE)** to defeat offline handshake cracking and dictionary attacks.
* **Passphrase Entropy Policy:** If forced to maintain WPA2-PSK networks, enforce long, high-entropy passphrases ($\ge 20$ alphanumeric/special characters) to mathematically defeat PBKDF2 wordlist attacks.
* **Enterprise Authentication (802.1X / EAP-TLS):** Replace shared keys (PSK) with RADIUS-backed 802.1X authentication utilizing client and server digital certificates (EAP-TLS).
* **Wireless Intrusion Prevention Systems (WIPS):** Deploy dedicated WIPS sensors to detect rogue APs, unexpected monitor mode cards, and automated deauthentication packet floods (`aireplay-ng --deauth`).
* **Management Frame Protection (MFP / 802.11w):** Enable IEEE 802.11w to cryptographically sign management frames, preventing unauthenticated Deauthentication/Disassociation denial-of-service attacks.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **BSSID vs SSID** | BSSID = 48-bit AP MAC address; SSID = Human-readable network name | BSSID identifies physical radio hardware; SSID identifies logical network |
| **Monitor Mode** | Allows NIC to passively capture all 802.11 frames on a channel | Does not require association with an AP |
| **4-Way Handshake** | Exchanged between AP and Client to establish encryption keys (PTK/GTK) | Capturing Messages 1-4 (or 1-2) enables offline WPA2 dictionary cracking |
| **Deauthentication Attack** | Sending forged 802.11 management frames (`0x000c`) to disconnect clients | Forces victim client to reconnect, allowing attacker to capture 4-way handshake |
| **PBKDF2 Calculation** | Hashes Passphrase + SSID 4,096 times to generate 256-bit PMK | Makes WPA2 offline cracking CPU-heavy; SSID acts as a salt |
| **Aircrack-ng** | Performs offline dictionary attacks against captured WPA/WPA2 handshakes | Attacks the passphrase strength, not the underlying AES-CCMP encryption |
| **IEEE 802.11w (MFP)** | Encrypts/signs 802.11 management frames | Protects networks against `aireplay-ng` deauthentication DoS attacks |
| **WPA3 Dragonfly / SAE** | Replaces PSK 4-way handshake with Zero-Knowledge Proof exchange | Eliminates susceptibility to offline dictionary cracking |
| **EAPOL Filter (`eapol`)** | Wireshark filter string used to isolate WPA/WPA2 handshake packets | Directly targets Extensible Authentication Protocol over LAN frames |
| **Radiotap Header** | Pseudo-header prepended to captured 802.11 frames | Conveys physical layer metadata (signal strength dBm, channel frequency) |