# Module 09 — Social Engineering

## Tier 1: Theoretical & Conceptual Foundations

### Baseline Definitions & Terminology
* **Social Engineering:** The art of manipulating individuals into performing specific actions or divulging confidential information by exploiting human psychology (trust, fear, urgency, curiosity, authority) rather than technical software vulnerabilities.
* **Credential Harvesting:** An attack vector where an adversary creates an exact replica of a trusted authentication web page to trick users into entering their credentials, capturing usernames and passwords in plaintext.
* **Phishing:** Mass, untargeted spam emails containing generic lures (e.g., "Invoice Overdue") sent broadly to harvest credentials or drop malware.
* **Spear Phishing:** Highly customized, targeted emails aimed at a specific individual, department, or role within an organization, crafted using gathered Open Source Intelligence (OSINT).
* **Whaling:** A high-impact variant of spear phishing targeting C-suite executives (CEO, CFO) or high-profile individuals to authorize unauthorized wire transfers or disclose sensitive corporate assets.
* **Clone Phishing:** The attacker intercepts or takes a legitimate, previously delivered email containing an attachment or link, creates an identical copy, and replaces the attachment/link with a malicious version.
* **Phishing Variations:**
  * **Vishing (Voice Phishing):** Social engineering over phone calls or VoIP using caller ID spoofing to extract PINs, passwords, or PII.
  * **Smishing (SMS Phishing):** Social engineering via Short Message Service (SMS) text messages containing malicious shortened links or call-back numbers.
  * **Pharming:** Compromising local host files or poisoning DNS cache records to invisibly redirect traffic destined for a legitimate website to an attacker-controlled IP address without user awareness.
  * **Angler Phishing:** Attackers register fraudulent social media customer service accounts (e.g., `@Ask_BankSupport`) to intercept dissatisfied customers posting complaints to legitimate brands.

### Psychological & Behavioral Human Triggers
Attackers manipulate six fundamental human psychological drivers to bypass rational decision-making:
* **Authority:** Impersonating executives, law enforcement, or IT administrators to demand compliance (e.g., "VP ordered this immediately").
* **Urgency:** Creating fake time limits (e.g., "Account suspended within 2 hours") to trigger panic and force rapid action before critical evaluation.
* **Scarcity:** Offering limited access to exclusive resources, promotions, or bonuses.
* **Social Proof / Consensus:** Convincing a victim that "everyone else in your department has already submitted this form."
* **Liking / Rapport:** Building friendly rapport through shared interests, common ground, or flattery prior to dropping the lure.
* **Reciprocity:** Granting a small favor (e.g., holding open a door or giving a free gift) so the victim feels obligated to return a favor (e.g., granting physical building entry).

---

## Tier 2: Attack Mechanics & Physical Vectors

### Physical Social Engineering Attack Vectors
* **Baiting:** Leaving physical media (USB drives, external SSDs) pre-loaded with autorun/LNK malicious payloads in public corporate areas (parking lots, cafeterias) with seductive labels (e.g., "Executive Compensation Q4.xlsx").
* **Tailgating vs. Piggybacking:**
  * **Tailgating:** Following an authorized individual through a secure electronic badge access point *without* their explicit knowledge or consent (e.g., slipping through before the door latches).
  * **Piggybacking:** Following an authorized person through a door *with* their explicit consent, usually obtained by manipulating courtesy (e.g., carrying heavy boxes and asking the victim to hold the door open).
* **Dumpster Diving:** Searching discarded corporate trash bins for non-shredded documents containing sensitive metadata, network diagrams, IP addresses, org charts, or passwords.
* **Shoulder Surfing:** Direct or remote observation (using cameras/binoculars) of a target's screen or keyboard to steal PINs, passwords, or sensitive data.
* **Eavesdropping:** Secretly listening to private audio conversations in hallways, breakrooms, elevators, or public coffee shops.
* **Diversion Theft:** Intercepting a delivery driver or courier prior to reaching a corporate facility and tricking them into dropping off shipments at an alternate location controlled by the attacker.
* **Watering Hole Attack:** Identifying websites frequently visited by a specific targeted group (e.g., defense contractors visiting an industry forum), compromising that specific third-party site, and injecting drive-by download malware.

### Impersonation & Advanced Attack Methodologies
* **Pretexting:** Inventing an elaborate, fictional scenario (a "pretext") where the attacker acts out a specific persona (e.g., internal auditor, police officer, vendor support) to establish a legitimate reason to request sensitive data.
* **Quid Pro Quo:** Offering an explicit service or benefit in exchange for information (e.g., calling employee extensions posing as "IT Helpdesk" offering to fix a system bug if the user provides their password).
* **Reverse Social Engineering:** The attacker intentionally creates a technical problem (e.g., causing a localized DoS on a client workstation), advertises themselves as the sole technical support solution, and waits for the victim to initiate contact and ask for assistance.
* **Tabnabbing:** A web-based attack where an open browser tab left idle automatically reloads and replaces its contents with a fake login page, tricking the returning user into re-authenticating.
* **Typosquatting / URL Hijacking:** Registering domains visually similar to target brands or common misspellings (e.g., `paypaI.com` using a capital `I` or `paypall.com`) to catch users who miskey web addresses.
* **AI-Assisted Social Engineering:** Using Large Language Models (LLMs) to automatically construct context-aware, grammatically flawless, and hyper-personalized phishing lures at scale, bypassing traditional language-based email filters.

---

## Tier 3: Tool Matrix & Technical Execution

### The Social-Engineer Toolkit (SET)
An open-source Python framework developed by Dave Kennedy (TrustedSec) for automated social engineering assessments.

```text
============================================================
           Social-Engineer Toolkit (SET) Core Menu
============================================================
1) Social-Engineering Attacks
2) Penetration Testing (Fast-Track)
3) Third-Party Modules
4) Update the Social-Engineer Toolkit
5) Update SET configuration
```

#### Primary SET Attack Modules
1. **Spear-Phishing Attack Vectors:** Sends mass targeted emails with embedded malicious file attachments (e.g., PDF/Word documents containing shellcode execution vectors).
2. **Website Attack Vectors:**
   * **Java Applet Attack Method:** Serves a signed/unsigned malicious Java applet prompting the user for execution rights to spawn a reverse shell.
   * **Metasploit Browser Exploit Method:** Auto-scans target browsers and serves matched browser exploits.
   * **Credential Harvester Attack Method:** Clones a target authentication page and runs a local web server to capture POST request parameters containing plaintext usernames and passwords. Sub-options include:
     * *Web Templates:* Built-in pre-made login pages.
     * *Site Cloner:* Clones any live target login portal automatically.
     * *Custom Import:* Imports custom HTML login templates.
   * **Tabnabbing:** Serves an idle tab replacement page.
3. **Mass Mailer Attack:** Integrates with local Sendmail or external SMTP relay servers to blast customized phishing templates with disguised hyperlinks or attachments to target lists.

### Phishing Detection & Analysis Tools
* **Netcraft:** Browser extension and web service used to analyze website domain age, hosting provider details, SSL certificate legitimacy, and block known phishing domains.
* **GoPhish:** Open-source phishing framework used by ethical hackers to conduct automated, scheduled enterprise phishing campaigns and track user click-through rates.

---

## Tier 4: Defense & Countermeasure Matrix

### Defensive Technical Controls (Email Security Protocols)
* **SPF (Sender Policy Framework):** Uses DNS `TXT` records to explicitly specify which IP addresses or mail servers are authorized to send email on behalf of a domain.
* **DKIM (DomainKeys Identified Mail):** Uses public-key cryptography to append a digital signature to outgoing email headers, allowing receiving servers to verify message integrity and sender domain authenticity.
* **DMARC (Domain-based Message Authentication, Reporting, and Conformance):** Builds on SPF and DKIM using DNS `TXT` records to define administrative actions (`none`, `quarantine`, `reject`) when emails fail SPF or DKIM checks, and generates aggregate failure reports.

### Physical & Procedural Safeguards
* **Mantraps / Air Lock Portals:** Physical security entryways featuring two interlocking doors where the first door must close completely before the second door unlocks, physically preventing tailgating.
* **Security Awareness Training & Phishing Simulations:** Continuous training programs paired with regular simulated phishing exercises to build organizational resistance against human manipulation.
* **Clean Desk Policy:** Mandating that all sensitive printed media, whiteboards, notes, and removable USB storage are locked away when workstations are left unattended.
* **Dual-Authorization Financial Controls:** Requiring two independent authorized approvals for out-of-band wire transfers, bank account changes, or executive credential resets.
* **Multi-Factor Authentication (MFA):** Mandatory enforcement of FIDO2/WebAuthn hardware security keys, ensuring harvested credentials alone cannot grant access to enterprise assets.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

### Scenario & Definition Triggers
| Question Stem Keyword / Scenario | Correct Answer / Concept |
| :--- | :--- |
| Drops flash drives in parking lot / "Executive Salaries" label | **Baiting** |
| Following someone through a door **WITHOUT** consent/knowledge | **Tailgating** |
| Following someone through a door **WITH** consent (holding heavy boxes) | **Piggybacking** |
| Digging through corporate trash bins for non-shredded diagrams/IPs | **Dumpster Diving** |
| Watching someone type a password using binoculars/cameras | **Shoulder Surfing** |
| Listening to private conversations in elevators/breakrooms | **Eavesdropping** |
| Intercepting a delivery driver to redirect shipments | **Diversion Theft** |
| Target company CFO/CEO targeted with custom high-value wire transfer lure | **Whaling** |
| Targeted email aimed at a specific individual or role using OSINT | **Spear Phishing** |
| Attacker creates a fake Twitter/X support handle (`@Ask_BankSupport`) | **Angler Phishing** |
| Social engineering over voice calls / VoIP / caller ID spoofing | **Vishing** |
| Social engineering via SMS / text messages with malicious links | **Smishing** |
| DNS cache or local hosts file modified to redirect traffic invisibly | **Pharming** |
| Attacker compromises a specific website frequently visited by targets | **Watering Hole Attack** |
| Attacker creates a problem, advertises help, and waits for victim to call | **Reverse Social Engineering** |
| Offering a service/favor in exchange for credentials ("IT reset") | **Quid Pro Quo** |
| Idle open browser tab automatically reloads into a fake login screen | **Tabnabbing** |
| Registering domains like `paypaI.com` (capital `I`) or `paypall.com` | **Typosquatting / URL Hijacking** |

### Social-Engineer Toolkit (SET) Menu Hooks
| Tool Selection Sequence / Feature | Exam Target |
| :--- | :--- |
| `Website Attack Vectors` $\rightarrow$ Clones target portal automatically | **Site Cloner** |
| `Website Attack Vectors` $\rightarrow$ Uses built-in pre-made login pages | **Web Templates** |
| `Website Attack Vectors` $\rightarrow$ Captures plaintext usernames/passwords | **Credential Harvester** |
| Sends mass emails with embedded malicious PDF/Word attachments | **Spear-Phishing Attack Vectors** |
| Sends mass emails using local Sendmail or external SMTP relay | **Mass Mailer Attack** |

### Email Defense & Protocol Hooks
| Protocol | Key Technical Function / Exam Hook |
| :--- | :--- |
| **SPF** | Uses DNS `TXT` records to specify **which IPs/servers can send mail** for a domain. |
| **DKIM** | Adds a **cryptographic digital signature** to email headers to prove integrity. |
| **DMARC** | Defines policy actions (**`none`**, **`quarantine`**, **`reject`**) for SPF/DKIM failures. |

### Physical & Procedural Defense Hooks
| Defense Mechanism | Primary Function / Trigger |
| :--- | :--- |
| **Mantraps / Air Lock Portals** | Two interlocking doors that physical lock to **prevent tailgating**. |
| **Clean Desk Policy** | Mandates locking away physical paper, USBs, and notes when stepping away. |
| **FIDO2 / Hardware Keys** | The ultimate MFA countermeasure to render **harvested credentials useless**. |