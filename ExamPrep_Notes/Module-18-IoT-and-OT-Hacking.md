# Module 18: IoT and OT Hacking

## Tier 1: Core Concepts & Principles

### IoT and OT Ecosystem Architecture
* **Internet of Things (IoT):** A network of physical devices embedded with electronics, software, sensors, and network connectivity, allowing them to collect, exchange, and process data.
  * *3-Tier IoT Architecture:* Edge Devices/Peripherals $\rightarrow$ IoT Gateway $\rightarrow$ Cloud Backend/Application Tier.
* **Operational Technology (OT) / Industrial Control Systems (ICS):** Hardware and software that monitors and controls physical devices, processes, and infrastructure in industrial settings.
  * *Purdue Model for ICS/OT:*
    * **Level 0 (Physical Process):** Sensors, actuators, physical devices.
    * **Level 1 (Basic Control):** Programmable Logic Controllers (PLCs), Remote Terminal Units (RTUs).
    * **Level 2 (Area Control):** Human-Machine Interfaces (HMIs), Supervisory Control and Data Acquisition (SCADA) software.
    * **Level 3 (Site Operations):** Historians, Domain Controllers, Jump Hosts.
    * **Level 4/5 (Enterprise Business):** Business networks, ERP servers, Corporate IT.
* **Core Vulnerability Profiles:** Limited hardware compute resources prevent heavy cryptographic implementations, default/hardcoded credentials, unencrypted communication protocols (MQTT, Modbus, CAN Bus), lack of firmware integrity checks, and direct Internet exposure without perimeter isolation.

---

## Tier 2: Technical Analysis & Mechanics

### IoT & OT Protocol Mechanics

#### 1. MQTT (Message Queuing Telemetry Transport)
* **Architecture:** Lightweight, ISO-standard publish/subscribe messaging protocol running over TCP (Default Ports: `1883` unencrypted, `8883` TLS).
* **Components:**
  * **MQTT Broker:** Central server that receives published messages and routes them to subscribed clients based on topic strings.
  * **Publishers/Subscribers:** IoT endpoints that transmit (`PUBLISH`) or listen (`SUBSCRIBE`) to specific topics (e.g., `factory/temp/sensor1`).
* **Vulnerability Mechanics:** Default deployments lack authentication and encryption, allowing attackers to connect to port 1883, subscribe to wildcard topics (`#`), sniff sensitive telemetry, or inject malicious control commands (`PUBLISH`).

#### 2. CAN Bus (Controller Area Network)
* **Architecture:** Robust vehicle bus standard allowing Electronic Control Units (ECUs) to communicate without a host computer (automotive & industrial robotics).
* **Packet Structure:** Standard CAN frames carry an **11-bit Identifier** (defines message priority/function), a **Data Length Code (DLC)**, and a **0 to 8-byte Payload**.
* **Vulnerability Mechanics:** Complete absence of authentication, addressing, or encryption. Any node on the physical or virtual CAN interface can passively sniff all frame traffic, forge packets, or execute **Replay Attacks** (`canplayer`) to duplicate physical actions (e.g., unlocking doors, acceleration).

---

## Tier 3: CLI, Tools & Framework Syntax

### Core IoT/OT Reconnaissance & Attack Utilities
* **Shodan / Censys:** Search engines for Internet-connected devices, indexing exposed IoT daemons, industrial control protocols (Modbus, BACnet, Siemens S7, MQTT), and banner metadata.
* **Wireshark:** Packet capture tool equipped with native dissectors for MQTT, Modbus, DNP3, and CAN Bus traffic.
* **can-utils:** Linux socketCAN utility suite for capturing, logging, generating, and replaying CAN network traffic.
* **ICSim (Instrument Cluster Simulator):** Software suite for simulating automotive CAN Bus networks.

### Key Commands & Execution Syntax

```bash
# Shodan Search Queries for Exposed IoT/OT Systems
# Find exposed unencrypted MQTT Brokers
port:1883

# Find exposed SCADA/ICS Modbus endpoints
port:502

# Find exposed Siemens S7 PLCs
port:102

# Google Dorks for Exposed SCADA/HMI Web Portals
inurl:"/view/view.shtml"
intitle:"Supervisory Control and Data Acquisition"
intext:"SCADA Login"

# Linux CAN Bus Interface Setup (can-utils)
# Bring up virtual CAN interface vcan0
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Sniff and Display Live CAN Traffic on vcan0
candump vcan0

# Capture and Log CAN Traffic to File for Replay Attack
candump -l vcan0

# Replay Captured CAN Traffic Log File onto CAN Bus (canplayer)
canplayer -I candump-2026-08-12_224320.log

# Inject Custom CAN Frame (ID: 0x244, Payload: 00 00 00 01)
cansend vcan0 244#00000001

# Wireshark Filters for IoT Traffic
# Filter for MQTT Protocol Traffic
mqtt

# Filter for MQTT Publish Messages Only
mqtt.msgtype == 3
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Defensive Architecture & Hardening Controls
* **Network Segmentation & Purdue Model Enforcement:**
  * Strict DMZ placement between Level 3 (OT Operations) and Level 4 (Enterprise IT).
  * Use Industrial Firewalls (deep packet inspection for Modbus/DNP3) to block unauthorized cross-zone commands.
* **MQTT Security Hardening:**
  * Mandate TLS encryption on Port 8883 (`MQTTS`).
  * Disable anonymous logins; enforce X.509 client certificate authentication and strict topic-level Access Control Lists (ACLs).
* **Automotive & CAN Bus Defense:**
  * Implement **SecOC (Secure On-Board Communication)** adding Message Authentication Codes (MAC) to CAN frames to neutralize replay and injection attacks.
  * Deploy automotive Security Gateways (Central Gateways) to isolate critical powertrain CAN buses from infotainment/telematics interfaces.
* **IoT Firmware Management:**
  * Remove hardcoded SSH/Telnet credentials from embedded Linux images.
  * Enforce Cryptographic Code Signing for over-the-air (OTA) firmware updates to prevent malicious flashing.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **MQTT Port 1883 vs 8883** | Port 1883 = Cleartext MQTT; Port 8883 = Encrypted MQTTS (TLS) | Standard lightweight IoT publish/subscribe messaging ports |
| **MQTT Wildcard (`#`)** | Multi-level wildcard subscription topic string | Subscribing to `#` forces the broker to send ALL published topic messages to the attacker |
| **CAN Bus Replay Attack** | Capturing frames via `candump -l` and re-injecting them via `canplayer` | Exploit possible due to zero built-in authentication or frame integrity checks |
| **Shodan `port:1883`** | Search string to discover publicly exposed MQTT brokers | Primary OSINT tool for identifying unprotected IoT infrastructure |
| **Purdue Model Level 1** | Identifies PLCs and RTUs executing direct physical control | Level 0 = Physical Sensors; Level 1 = Control Logic; Level 2 = HMI/SCADA |
| **`candump` vs `canplayer`** | `candump` logs CAN frames; `canplayer` replays logged CAN files | `can-utils` command pairing for automotive traffic auditing |
| **Modbus Port 502** | Default TCP port for industrial Modbus communication | Legacy SCADA protocol completely lacking authentication mechanisms |
| **Siemens S7 Port 102** | Default TCP port for Siemens S7 PLC communication | Key industrial target port queried in Shodan/Censys scans |
| **SecOC (Secure On-Board Communication)** | Defensive technology adding MAC integrity to CAN frames | Primary countermeasure against CAN Bus replay and injection attacks |
| **Google Dorking for SCADA** | Using targeted search parameters (`inurl:`, `intitle:`) to locate HMIs | Uncovers exposed web-based SCADA login panels indexed by search engines |