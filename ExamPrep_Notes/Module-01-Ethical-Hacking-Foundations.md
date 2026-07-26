# Module 01: Ethical Hacking Foundations

## 1.1 Information Security Concepts & Elements

### Core Security Principles
* **Information Security (InfoSec):** Protection of information and information systems from unauthorized access, use, disclosure, disruption, modification, or destruction.
* **The CIA Triad:**
  * **Confidentiality:** Ensures data is accessible only to authorized individuals.
    * *Controls:* Encryption (AES, RSA), Access Control Lists (ACLs), Data Classification, Multi-Factor Authentication (MFA).
  * **Integrity:** Guarantees data accuracy and guards against unauthorized modification or deletion.
    * *Controls:* Cryptographic Hashing (SHA-256, SHA-3), Digital Signatures, Version Control, Checksums.
  * **Availability:** Ensures timely and reliable access to resources for authorized users.
    * *Controls:* High Availability (HA), Load Balancing, Disaster Recovery (DR), Data Backups, DDoS Mitigation.

### Essential Security Terminology
* **Authenticity:** Verification that a user, message, or system is genuine and holds valid credentials.
* **Non-Repudiation:** Proof of origin and integrity of data ensuring a sender cannot deny sending a message (achieved via Public Key Infrastructure and Digital Signatures).
* **Vulnerability:** Weakness or flaw in software, hardware, or system procedures that can be exploited.
* **Threat:** Any potential cause of an unwanted incident that could harm a system or organization.
* **Exploit:** Executable code, script, or sequence of commands that takes advantage of a vulnerability.
* **Zero-Day Attack:** Attack that exploits an unknown software vulnerability before the vendor releases a patch.
* **Payload:** The part of the exploit code or malware that performs the malicious action on the target host.

### Risk Management
* **Risk Definition:** Probability of a threat exploiting a vulnerability and the resulting business impact.
* **Risk Formula:** $\text{Risk} = \text{Threat} \times \text{Vulnerability} \times \text{Impact}$
* **Risk Response Strategies:**
  * **Risk Avoidance:** Eliminating the risk entirely by removing the vulnerable asset or activity.
  * **Risk Mitigation:** Implementing security controls to reduce likelihood or impact.
  * **Risk Transfer:** Shifting risk responsibility to a third party (e.g., Cyber Insurance).
  * **Risk Acceptance:** Acknowledging and absorbing the risk when mitigation costs exceed potential loss.

---

## 1.2 Cyber Attack Vectors & Threat Intelligence

### Primary Attack Vectors
* **Social Engineering:** Manipulating individuals into divulging confidential information (e.g., Phishing, Vishing, Spear Phishing, Baiting).
* **Malware Attacks:** Infiltrating systems using malicious software (Trojans, Ransomware, Keyloggers, Rootkits).
* **Drive-by Downloads:** Unintentional download of malicious code initiated by visiting a compromised website.
* **Insider Threats:** Malicious or careless activities performed by individuals with legitimate internal access.
* **Supply Chain Attacks:** Exploiting vulnerabilities within third-party vendors, suppliers, or software dependencies.
* **Advanced Persistent Threats (APTs):** Long-term, highly targeted, stealthy cyberattacks executed by well-funded adversaries.

### Cyber Threat Intelligence (CTI)
* **Strategic Intelligence:** High-level information regarding business risks, adversary trends, and financial motives (targeted at C-suite Executives).
* **Tactical Intelligence:** Information on adversary Tactics, Techniques, and Procedures (TTPs) (targeted at Security Operations Centers).
* **Operational Intelligence:** Specific details regarding incoming attacks, campaigns, or threat actor profiles (targeted at Security Analysts).
* **Technical Intelligence:** Indicators of Compromise (IoCs) like IP addresses, malicious URLs, file hashes, and domain names (targeted at Automated Security Systems).

---

## 1.3 Threat Actor Classifications

### Actor Types & Characteristics
* **Black Hat:** Driven by financial gain, espionage, or malice; high technical skills; operates illegally without authorization.
* **White Hat:** Security professionals acting defensively to secure systems; high technical skills; operates with full authorization.
* **Gray Hat:** Driven by curiosity or unauthorized security evaluation; moderate to high skills; acts without explicit authorization but lacks malicious intent.
* **Script Kiddies:** Driven by thrill-seeking or personal notoriety; low technical skills; relies on pre-built scripts and tools.
* **Hacktivists:** Driven by political, social, or ideological causes; moderate technical skills; uses defacement, DDoS, and leaks.
* **State-Sponsored / APT:** Driven by nation-state espionage, military operations, or strategic advantage; very high technical skills; backed by government resources.
* **Insider Threat:** Driven by revenge, financial gain, or carelessness; varying technical skills; possesses legitimate internal access.

---

## 1.4 Attack Models & Frameworks

### Cyber Kill Chain (Lockheed Martin)
1. **Reconnaissance:** Gathering intelligence on the target (emails, open ports, IP ranges).
2. **Weaponization:** Coupling an exploit payload with a deliverable file (e.g., PDF/Docx with executable payload).
3. **Delivery:** Transmitting the weaponized payload to the victim (e.g., via Phishing email, malicious link).
4. **Exploitation:** Executing code on the victim system by exploiting software or human vulnerabilities.
5. **Installation:** Establishing persistent access on the target (installing Backdoors, Trojans, Web Shells).
6. **Command and Control (C2):** Establishing a covert communication channel back to the attacker's server.
7. **Actions on Objectives:** Executing the final attack objective (Data Exfiltration, Encryption, Sabotage).

### MITRE ATT&CK Framework
* **Tactics:** The high-level operational goal of the adversary during an attack phase (the *Why*).
  * *Examples:* Initial Access, Execution, Persistence, Privilege Escalation, Defense Evasion, Credential Access, Discovery, Lateral Movement, Collection, Exfiltration, Impact.
* **Techniques:** The specific method or action used to achieve a tactical goal (the *How*).
  * *Example:* T1059 (Command and Scripting Interpreter).
* **Sub-Techniques:** Detailed categorization of a specific technique mechanism.
  * *Example:* T1059.001 (PowerShell).

---

## 1.5 Ethical Hacking & Penetration Testing Methodologies

### Types of Security Assessments
* **Vulnerability Assessment:** Identifies, quantifies, and prioritizes vulnerabilities without actively exploiting them.
* **Penetration Testing:** Active execution of attack vectors to exploit vulnerabilities and measure actual system resilience.

### Penetration Testing Approaches
* **Black-Box Testing:** No prior knowledge of internal target infrastructure (simulates external threat actor).
* **White-Box Testing:** Complete internal knowledge provided, including source code, network diagrams, and IP mappings (simulates internal developer/admin audit).
* **Gray-Box Testing:** Partial knowledge provided (e.g., standard low-privileged user account, internal IP subnets).

### Phases of Penetration Testing
1. **Pre-engagement & Scoping:** Defining Rules of Engagement (RoE), scope parameters, legal authorizations, testing windows, and emergency points of contact.
2. **Information Gathering / Reconnaissance:** Passive and active OSINT data gathering.
3. **Vulnerability Analysis:** Scanning target assets for unpatched flaws or misconfigurations.
4. **Exploitation:** Executing authorized attacks to gain access.
5. **Post-Exploitation:** Maintaining access, escalating privileges, lateral movement, and assessing impact.
6. **Reporting & Remediation:** Documenting findings, PoCs, risk ratings (CVSS), and actionable remediation steps.

---

## 1.6 Information Security Laws, Regulations & Compliance Standards

### Major Security & Privacy Legislation
* **Computer Fraud and Abuse Act (CFAA) (US):** Primary federal statute prohibiting unauthorized access to computers and protected systems.
* **Electronic Communications Privacy Act (ECPA) (US):** Restricts unauthorized interception of electronic wire and oral communications.
* **General Data Protection Regulation (GDPR) (EU):** Standard regulating data privacy, consumer consent, and protection of Personally Identifiable Information (PII) for EU citizens.
* **Health Insurance Portability and Accountability Act (HIPAA) (US):** Mandates data security standards for Protected Health Information (PHI) in healthcare systems.
* **Sarbanes-Oxley Act (SOX) (US):** Mandates strict financial recordkeeping and internal controls auditing for public companies to prevent fraud.

### Industry Compliance Standards
* **PCI-DSS (Payment Card Industry Data Security Standard):** Requirements governing organizations processing, storing, or transmitting credit card and payment data.
* **ISO/IEC 27001:** Global standard defining requirements for establishing, implementing, maintaining, and continually improving an Information Security Management System (ISMS).
* **NIST SP 800-53:** Comprehensive catalog of security controls for federal information systems and organizations.