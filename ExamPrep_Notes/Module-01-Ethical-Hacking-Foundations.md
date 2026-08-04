# Module 01: Ethical Hacking Foundations

## Tier 1: Core Concepts & Definitions
* **Confidentiality:** Assures that information and system resources are accessible only to authorized entities, preventing unauthorized access or disclosure.
* **Integrity:** Guarantees the trustworthiness, accuracy, and completeness of data by safeguarding it against unauthorized modification, deletion, or tampering.
* **Availability:** Ensures that systems, networks, and applications are continuously accessible and operational for authorized users when needed.
* **Authenticity:** Confirms the identity of a user, process, system, or data origin before granting access or establishing trust.
* **Non-Repudiation:** Provides cryptographic and logging proof of data transmission or actions performed, preventing an entity from denying its involvement.
* **Vulnerability Assessment:** Scans, identifies, and categorizes technical weaknesses and misconfigurations in a system or network **without** attempting exploitation.
* **Penetration Testing:** Actively verifies security posture by exploiting identified vulnerabilities to determine the real-world operational impact and breach depth.
* **Black-Box Testing:** Testing performed with zero prior internal knowledge of the target environment, simulating an external adversary relying heavily on OSINT.
* **White-Box Testing:** Testing performed with complete internal access to architecture documentation, source code, network diagrams, and system configurations.
* **Gray-Box Testing:** Testing performed with partial knowledge, such as non-privileged user access or internal subnet ranges, simulating an insider threat or compromised account.

---

## Tier 2: Deep-Dive Technical Analysis & Frameworks

### Security Control Classifications
* **Preventive:** Blocks unauthorized access, attacks, or security policy violations before execution (e.g., Firewalls, IPS, Access Control Lists, MFA).
* **Detective:** Identifies, logs, and alerts on malicious activity during or immediately after execution (e.g., IDS, SIEM correlation rules, EDR, Audit Logging).
* **Corrective:** Restores system posture, mitigates damage, and repairs configurations post-incident (e.g., Patch management deployment, system re-imaging, automated script isolation).
* **Deterrent:** Discourages malicious actors from attempting an attack through psychological or policy warnings (e.g., Legal warning banners, visible CCTV).
* **Compensating:** Alternative security control implemented when primary controls are technically or operationally unfeasible (e.g., Network microsegmentation for legacy unpatchable systems).
* **Recovery:** Restores essential business operational capabilities following a major security incident or disaster (e.g., DR site failover, backup restores).

### Risk Assessment & Cost-Benefit Formulas
* **Single Loss Expectancy (SLE):** Financial loss incurred each time a specific risk event occurs.
  $$\text{SLE} = \text{Asset Value (AV)} \times \text{Exposure Factor (EF)}$$
* **Annualized Loss Expectancy (ALE):** Projected financial loss for a specific risk over a one-year period.
  $$\text{ALE} = \text{SLE} \times \text{Annualized Rate of Occurrence (ARO)}$$
* **Cost-Benefit Analysis (CBA):** Determines if a safeguard is financially justified.
  $$\text{Safeguard Value} = \text{ALE}_{\text{prior}} - (\text{ALE}_{\text{post}} + \text{Annual Cost of Control})$$
  * *Decision Rule:* If $\Delta\text{ALE} > \text{Annual Control Cost}$, deploy safeguard; otherwise, select **Risk Acceptance**.

### Threat Modeling Frameworks
* **STRIDE (Microsoft):** Threat-centric categorizations mapped to security principles:
  * **S**poofing Identity $\rightarrow$ Impacts *Authenticity*.
  * **T**ampering with Data $\rightarrow$ Impacts *Integrity*.
  * **R**epudiation $\rightarrow$ Impacts *Non-Repudiation*.
  * **I**nformation Disclosure $\rightarrow$ Impacts *Confidentiality*.
  * **D**enial of Service $\rightarrow$ Impacts *Availability*.
  * **E**levation of Privilege $\rightarrow$ Impacts *Authorization*.
* **PASTA:** Risk-centric 7-stage methodology directly aligning technical vulnerabilities with operational business objectives.
* **DREAD:** Vulnerability prioritization scoring system based on average rating across: **D**amage, **R**eproducibility, **E**xploitability, **A**ffected Users, **D**iscoverability.

### Threat Intelligence & Framework Mechanics
* **STIX & TAXII:** STIX (Structured Threat Information eXpression) defines the JSON schema language (**WHAT** threat data is). TAXII (Trusted Automated eXchange of Indicator Information) is the HTTPS transport protocol (**HOW** threat data is delivered).
* **Cyber Kill Chain (Lockheed Martin):** Reconnaissance $\rightarrow$ Weaponization $\rightarrow$ Delivery $\rightarrow$ Exploitation $\rightarrow$ Installation $\rightarrow$ C2 $\rightarrow$ Actions on Objectives.
  * *Critical Exam Distinctions:* **Weaponization** occurs purely on *Attacker infrastructure*. **Exploitation** occurs on the *Victim host*.
* **MITRE ATT&CK Framework:** Structured matrix organizing real-world threat actor behaviors:
  * **Tactics:** High-level adversary objectives (*Why* an action is taken).
  * **Techniques:** Specific methods to achieve tactical goals (*How* it is executed).
  * **Sub-Techniques / Procedures:** Specific software commands or execution scripts used in real-world campaigns.

---

## Tier 3: Command-Line Syntax & Tool Execution

* **Checking Rules of Engagement / Scope Metrics:**
  * Active penetration testing probes transmitted without signed legal authorization explicitly violate statutes like the CFAA (18 U.S.C. § 1030).
* **STIX Payload Processing Syntax Structure:**
  * Interacting with threat feeds over TAXII endpoints:
  ```bash
  # Querying a TAXII server discovery endpoint via cURL
  curl -X GET [https://cti-server.example.com/taxii2/](https://cti-server.example.com/taxii2/) -H "Accept: application/taxii+json;version=2.1"
  ```
* **Calculating Quantitative Metrics in Scripts:**
  ```bash
  # Quick bash calculation for ALE
  python3 -c "AV=100000; EF=0.3; ARO=0.5; SLE=AV*EF; ALE=SLE*ARO; print(f'SLE: {SLE}, ALE: {ALE}')"
  ```

---

## Tier 4: Real-World Scenario Blueprints

### Scenario 1: Medical Device Network Isolation
* **Context:** A hospital operates legacy embedded operational technology (OT) hosting unpatchable vulnerabilities that cannot be updated without voiding regulatory compliance.
* **Analysis:** Primary preventive controls (patching) are unviable. The team deploys strict VLAN microsegmentation and deep packet inspection (DPI) firewalls to monitor traffic.
* **Execution/Verdict:** This is a classic implementation of a **Compensating Control**.

### Scenario 2: Cloud Hypervisor Context Breakout
* **Context:** A security analyst discovers a flaw in a shared public cloud instance where an attacker can break out of a restricted guest container/VM to execute arbitrary code on the hypervisor host.
* **Analysis:** Under CVSS v3.1 rating metrics, the vulnerability impacts components beyond its immediate security authorization scope.
* **Execution/Verdict:** The CVSS metric for **Scope (S)** shifts from **Unchanged (U)** to **Changed (C)**, significantly increasing the overall severity score.

### Scenario 3: Legal Liability During Third-Party Assessments
* **Context:** A penetration tester identifies an active remote code execution vulnerability on a client's core production database during an off-hours assessment window.
* **Analysis:** The tester must act in full compliance with the agreed **Rules of Engagement (RoE)** and **Scope of Work (SoW)** without exceeding authorized boundaries.
* **Execution/Verdict:** The tester must immediately halt active exploitation on that host, document findings, and notify designated client emergency contacts specified in the RoE before proceeding.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Key Hook / Exam Trigger | Critical Distinction / Technical Trap |
| :--- | :--- | :--- |
| **Confidentiality** | Protection against unauthorized disclosure | Enforced via AES, RSA, ACLs, MAC, MFA |
| **Integrity** | Protection against unauthorized data modification | Enforced via SHA-256, HMAC, Digital Signatures, FIM |
| **Availability** | Continuous, timely operational access | Enforced via HA clusters, RAID, Load Balancers, Backups |
| **Authenticity** | Verification of origin identity and legitimacy | Enforced via PKI certificates, FIDO2/WebAuthn |
| **Non-Repudiation** | Proof of origin; sender cannot deny sending | Private key signature + immutable centralized logging |
| **Preventive Control** | Proactively stops an attack before execution | Firewalls, IPS, ACLs, MFA |
| **Detective Control** | Identifies and alerts on active security incidents | IDS, SIEM correlation rules, EDR |
| **Corrective Control** | Repairs damage and restores normal state | Re-imaging hosts, automated isolation, applying patches |
| **Deterrent Control** | Discourages attackers from attempting attacks | Legal warning banners, visible CCTV cameras |
| **Compensating Control**| Alternative safeguard when primary is unfeasible | Dedicated isolated VLAN + DPI for legacy systems |
| **Recovery Control** | Restores operational capacity post-disaster | DRP execution, DR site failover, backup restoration |
| **Single Loss Expectancy**| Financial loss per single threat event | $\text{SLE} = \text{Asset Value (AV)} \times \text{Exposure Factor (EF)}$ |
| **Annualized Rate** | Estimated yearly frequency of a threat | $\text{ARO} = 12.0$ (monthly), $\text{ARO} = 0.1$ (once every 10 yrs) |
| **Annualized Loss** | Total expected annual loss for a specific risk | $\text{ALE} = \text{SLE} \times \text{ARO}$ |
| **Cost-Benefit Analysis**| Validates financial viability of a control | Implement safeguard only if $\Delta\text{ALE} > \text{Annual Safeguard Cost}$ |
| **Risk Avoidance** | Completely eliminating risk by stopping activity | Discontinuing a legacy service or vulnerable application |
| **Risk Mitigation** | Implementing safeguards to lower risk level | Installing firewalls, deploying EDR, patching software |
| **Risk Transfer** | Offloading financial risk to a third party | Purchasing Cyber Insurance, using managed service provider |
| **Risk Acceptance** | Absorbing financial impact without controls | Chosen when safeguard cost exceeds potential risk loss |
| **STRIDE - Spoofing** | Impersonating legitimate user/system | Violates **Authenticity** |
| **STRIDE - Tampering** | Modifying data or execution parameters | Violates **Integrity** |
| **STRIDE - Repudiation**| Denying performed actions due to weak logs | Violates **Non-Repudiation** |
| **STRIDE - Info Disclosure**| Exposing sensitive data to unauthorized entities| Violates **Confidentiality** |
| **STRIDE - DoS** | Disrupting or halting service availability | Violates **Availability** |
| **STRIDE - Elevation** | Gaining unauthorized administrative privileges | Violates **Authorization** |
| **PASTA** | Risk-centric 7-stage threat modeling | Directly aligns technical threats with business objectives |
| **DREAD** | Risk prioritization scoring metric | Calculated by averaging **D**, **R**, **E**, **A**, **D** factors |
| **Script Kiddie** | Unskilled attacker using pre-built tools | Lacks custom exploit development ability |
| **Hacktivist** | Driven by political, social, or religious goals | Uses defacements, leaks, and DDoS attacks |
| **APT / State-Sponsored**| Skilled, highly funded nation-state threat | Long-term cyber espionage and strategic persistence |
| **Insider Threat** | Current/former employee misusing access | Dangerous due to existing internal trust and rights |
| **Strategic CTI** | Executive-level strategic threat insight | Focuses on business risk, overall trends, and finances |
| **Operational CTI** | SOC leadership and incident management | Focuses on incoming campaigns, motives, and attacker intent |
| **Tactical CTI** | Targeted at SOC analysts and threat hunters | Maps threat actor TTPs directly to MITRE ATT&CK |
| **Technical CTI** | Feeds for automated security appliances | Consists of IoCs (IPs, URLs, file hashes, domain names) |
| **STIX** | JSON-based data format for threat data | Defines **WHAT** threat data looks like (Schema) |
| **TAXII** | Protocol transporting STIX payloads over HTTPS | Defines **HOW** threat data is delivered |
| **Cyber Kill Chain** | Lockheed Martin 7-stage attack framework | Recon $\rightarrow$ Weaponization $\rightarrow$ Delivery $\rightarrow$ Exploitation $\rightarrow$ Installation $\rightarrow$ C2 $\rightarrow$ Actions |
| **Kill Chain: Weaponization**| Packaging exploit code into deliverable payload| **MUST** occur on attacker infrastructure, NOT victim |
| **Kill Chain: Exploitation**| Triggering exploit code against victim vulnerability| Occurs directly on target host environment |
| **MITRE ATT&CK** | Matrix organizing real-world adversary behavior | Tactics (**Why**), Techniques (**How**), Sub-techniques, Procedures |
| **Vulnerability Assessment**| Scans and categorizes system weaknesses | **NO EXPLOITATION** permitted; identifies surface flaws |
| **Penetration Testing** | Active verification of real-world risk level | **ACTIVELY EXPLOITS** weaknesses to validate impact |
| **Black-Box Testing** | Zero prior target architecture knowledge | Simulates external attacker; high reliance on OSINT |
| **White-Box Testing** | Full documentation, code, and network maps | Simulates internal developer or comprehensive security audit |
| **Gray-Box Testing** | Limited user credentials or internal ranges | Simulates standard insider threat or compromised account |
| **Rules of Engagement** | Explicit boundaries, allowed exploits, limits | **MUST** be signed by corporate C-level/Legal before scanning |
| **CFAA** | Primary U.S. federal anti-hacking statute | Prosecutes unauthorized computer access and damage |
| **GDPR** | EU personal data privacy regulatory standard | Mandates data breach notifications within **72 hours** |
| **PCI-DSS** | Compliance for handling payment card data | Requires quarterly ASV scans and annual security audits |
---

## 1.6 High-Yield CEH Exam Traps & Distinction Matrix

| Concept Pair | Critical Distinction for CEH Scenarios |
| :--- | :--- |
| **Vulnerability Assessment vs. Penetration Test** | Vulnerability Assessment identifies/categorizes flaws (**NO** exploitation allowed). Penetration Test actively **EXPLOITS** flaws to prove real-world vulnerability. |
| **Risk Mitigation vs. Risk Transfer** | Risk Mitigation deploys controls to reduce likelihood/impact. Risk Transfer offloads financial risk to a 3rd party (e.g., Cyber Insurance). |
| **STIX vs. TAXII** | STIX is the language/format defining threat data (JSON Schema). TAXII is the protocol/transport executing delivery over HTTPS. |
| **Weaponization vs. Exploitation (Kill Chain)** | Weaponization occurs entirely on **Attacker** infrastructure. Exploitation is executed on the **Victim** host upon payload execution. |

* **Rule of Engagement Authorization Trap:** Executing penetration testing probes without explicit, signed legal authorization from an authorized corporate officer is defined as unauthorized access under legal statutes like CFAA, regardless of tester intent.
* **Quantitative Risk Decisions:** Compute:
  $$\Delta\text{ALE} = \text{ALE}_{\text{prior}} - \text{ALE}_{\text{post}}$$
  If $\Delta\text{ALE} > \text{Annual Safeguard Cost}$, implement the control. If $\Delta\text{ALE} < \text{Annual Safeguard Cost}$, select **Risk Acceptance**.
* **Scope (S) Changes in CVSS v3.1:** If a vulnerability allows an attacker to break out of a restricted context to impact surrounding components (e.g., guest VM escape to host hypervisor), the Scope metric changes from **Unchanged (U)** to **Changed (C)**.