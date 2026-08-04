# Module 01: Ethical Hacking Foundations

## Tier 1: Core Concepts & Definitions
* **Confidentiality:** Ensures sensitive data is accessible only to authorized entities.
* **Integrity:** Ensures protection against unauthorized data modification, tampering, or deletion.
* **Availability:** Guarantees timely and reliable access to systems and services for authorized users.
* **Authenticity:** Verifies the genuine identity of a user, process, or system.
* **Non-Repudiation:** Ensures an action or message sender cannot deny their involvement (achieved via Digital Signatures and audit logs).
* **Vulnerability Assessment:** Scans and categorizes system weaknesses **without active exploitation**.
* **Penetration Testing:** Actively **exploits vulnerabilities** to evaluate real-world security impact.
* **Black-Box Testing:** Testing performed with zero prior knowledge of the target; simulates external attackers.
* **White-Box Testing:** Testing performed with full knowledge, source code, and system documentation provided.
* **Gray-Box Testing:** Testing performed with partial knowledge (e.g., standard user credentials); simulates insider threats.

---

## Tier 2: Core Technical Concepts & Frameworks

### Security Control Classifications
* **Preventive Control:** Blocks unauthorized access or malicious actions before they occur (e.g., Firewalls, IPS, Access Control Lists, MFA).
* **Detective Control:** Identifies, logs, and alerts on security events during or after occurrence (e.g., IDS, SIEM, Audit Logs).
* **Corrective Control:** Restores system state or fixes damage post-incident (e.g., Re-imaging systems, deploying patches).
* **Deterrent Control:** Discourages malicious attempts via legal or visual warnings (e.g., Legal banners, visible security cameras).
* **Compensating Control:** Alternative safeguard used when primary controls are technically or operationally unfeasible (e.g., Network microsegmentation for unpatchable legacy systems).
* **Recovery Control:** Restores operational capacity after a major disruption or disaster (e.g., DR site failover, restoring from clean backups).

### Risk Management Concepts
* **Risk Avoidance:** Completely eliminating risk by stopping or removing the risky activity.
* **Risk Mitigation:** Applying controls and safeguards to reduce risk to an acceptable level.
* **Risk Transfer:** Offloading financial risk to a third party (e.g., Cyber insurance, third-party MSP).
* **Risk Acceptance:** Choosing to absorb potential financial loss without deploying safeguards (used when control cost exceeds potential loss).

### Threat Modeling & Attack Frameworks
* **STRIDE Threat Framework:**
  * **Spoofing:** Fake identity $\rightarrow$ Violates *Authenticity*.
  * **Tampering:** Data alteration $\rightarrow$ Violates *Integrity*.
  * **Repudiation:** Denying actions $\rightarrow$ Violates *Non-Repudiation*.
  * **Information Disclosure:** Exposing data $\rightarrow$ Violates *Confidentiality*.
  * **Denial of Service:** Service disruption $\rightarrow$ Violates *Availability*.
  * **Elevation of Privilege:** Unauthorized access $\rightarrow$ Violates *Authorization*.
* **Cyber Kill Chain (Lockheed Martin):** Reconnaissance $\rightarrow$ Weaponization $\rightarrow$ Delivery $\rightarrow$ Exploitation $\rightarrow$ Installation $\rightarrow$ Command & Control (C2) $\rightarrow$ Actions on Objectives.
* **MITRE ATT&CK Matrix:**
  * **Tactics:** High-level operational goal (*Why* an attacker acts).
  * **Techniques:** Execution method (*How* the objective is achieved).
  * **Procedures:** Exact tool, script, or command used.

---

## Tier 3: Legal & Organizational Standards
* **Rules of Engagement (RoE):** Formal document signed by authority defining allowed scope, target IP ranges, testing windows, and restricted techniques before testing begins.
* **CFAA (Computer Fraud and Abuse Act):** Primary U.S. anti-hacking legislation prosecuting unauthorized access or exceeding authorized access.
* **GDPR:** EU data protection regulation requiring notification of personal data breaches to regulatory bodies within **72 hours**.
* **PCI-DSS:** Payment industry compliance standard requiring quarterly vulnerability scans by Authorized Scanning Vendors (ASVs) and annual security assessments.

---

## Tier 4: Exam Scenario Blueprints

### Scenario 1: Unpatchable System Security
* **Context:** An organization relies on legacy operational software that cannot be patched without breaking operational stability.
* **Analysis:** Primary preventive patching cannot be performed. The team isolates the server on a dedicated VLAN with strict inline inspection.
* **Verdict:** This setup represents a **Compensating Control**.

### Scenario 2: Scope Boundaries During Assessments
* **Context:** During an authorized penetration test, a tester finds a severe vulnerability on an IP address belonging to the client but not listed in the initial scope agreement.
* **Analysis:** Testing unlisted IPs violates legal agreements regardless of ownership.
* **Verdict:** Halt activity on that host immediately, document the vulnerability, and request explicit written authorization from client leadership before proceeding.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **Confidentiality** | Prevents unauthorized data access | Maintained via Encryption, ACLs, MFA |
| **Integrity** | Prevents unauthorized data modification | Maintained via Hashing, Digital Signatures |
| **Availability** | Guarantees operational system access | Maintained via Backups, Redundancy, DDoS Defenses |
| **Non-Repudiation** | Proves sender identity; cannot deny action | Digital Signatures + Centralized Audit Logs |
| **Preventive Control** | Blocks attack before execution | Firewalls, IPS, MFA, ACLs |
| **Detective Control** | Alerts on active or past attack activity | IDS, SIEM, System Audit Logs |
| **Corrective Control** | Repairs systems post-attack | Patching, Re-imaging infected systems |
| **Compensating Control**| Alternative control when primary fails/unusable| Isolated network VLANs for legacy systems |
| **Risk Avoidance** | Stop activity entirely to remove risk | Discontinuing a vulnerable service |
| **Risk Mitigation** | Reduce risk using controls | Applying patches, installing antivirus |
| **Risk Transfer** | Shift risk impact to external entity | Cyber Insurance |
| **Risk Acceptance** | Absorb risk when control cost > loss | Proceeding without controls when impact is trivial |
| **Script Kiddie** | Attacker using pre-built tools without deep skill| Lacks ability to write custom exploits |
| **Hacktivist** | Driven by political/social motives | Website defacement, DDoS, public data leaks |
| **APT / Nation-State**| Highly funded, sophisticated persistent threat | Focuses on long-term stealth and cyber espionage |
| **Insider Threat** | Employee or contractor abusing existing access | High risk due to existing trust and valid credentials |
| **Kill Chain: Weaponization**| Coupling exploit with payload | Occurs on **attacker infrastructure** |
| **Kill Chain: Exploitation**| Triggering payload code against target flaw | Occurs on **victim host** |
| **Vulnerability Assessment**| Identify and categorize weaknesses | **No exploitation** allowed |
| **Penetration Testing** | Validate weakness severity via exploit | **Active exploitation** performed |
| **Black-Box Test** | Zero target knowledge provided | Simulates external attacker |
| **White-Box Test** | Complete access and documentation provided | Simulates internal developer or full audit |
| **Gray-Box Test** | Partial access/user credentials provided | Simulates insider or compromised user account |
| **Rules of Engagement** | Formal legal limits and boundaries | Must be signed by executive leadership before testing |
| **GDPR Breach Warning**| Mandatory breach notification timeframe | **72 hours** |
| **PCI-DSS Scanning** | Mandatory scanning interval for card processors | **Quarterly** (every 3 months) by ASV |