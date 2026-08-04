# Module 02: Footprinting and Reconnaissance

## Tier 1: Core Concepts & Principles

### Principles of Footprinting
* **Footprinting Definition:** The systematic baseline step of ethical hacking where an attacker or penetration tester gathers detailed technical, operational, and organizational information about a target network before launching an attack.
* **Primary Objectives:**
  * Map network architecture, active host subnets, domain hierarchies, and administrative ownership.
  * Uncover vulnerable services, misconfigurations, and entry vectors to construct an explicit threat model.
  * Formulate an accurate target footprint without alerting network defenders during passive stages.
* **Passive vs. Active Footprinting Matrix:**

| Dimension | Passive Footprinting | Active Footprinting |
| :--- | :--- | :--- |
| **Direct Target Contact** | None (Interacts with third parties / public caches) | Direct (Sends packets directly to target endpoints) |
| **Detection Probability** | Extremely Low / Invisible to target IDS | High (Logged by target Firewalls, WAFs, and SIEM) |
| **CEH Exam Examples** | OSINT searches, WHOIS lookups, SHODAN, Social Media | DNS Zone Transfers, Nmap ping sweeps, Web Crawling |
| **Legal / Policy Impact** | Purely public information gathering | May breach strict rules of engagement if unauthorized |

---

## Tier 2: Technical Analysis & Reconnaissance Methodology

### Search Engine Reconnaissance & Google Dorking
* **Google Hacking Database (GHDB):** A public collection of search queries designed to expose sensitive files, exposed admin portals, database credentials, and vulnerable scripts indexed by web crawlers.
* **CEH High-Yield Google Dorks Reference:**

| Operator / Dork | Exam Function & Syntax | Practical Exam Target / Scenario |
| :--- | :--- | :--- |
| `site:` | Restricts results to a single domain (e.g., `site:target.com`) | Isolate corporate digital assets |
| `filetype:` | Filters by specific file extensions (e.g., `filetype:pdf` or `xls`) | Discover sensitive internal documentation |
| `inurl:` | Scans for strings in the page URL (e.g., `inurl:admin`) | Identify administrative portals and login endpoints |
| `intitle:` | Scans for text within the HTML `<title>` tag | Uncover directory listings (`intitle:"index of"`) |
| `intext:` | Scans body text of web pages (e.g., `intext:"sql syntax near"`) | Locate exposed error logs or credentials |
| `link:` | Finds web pages linking to a specific URL | Perform reverse link dependency mapping |
| `cache:` | Displays Google's stored snapshot of a webpage | View content deleted or hidden by the server |

### Web & Domain Reconnaissance
* **Website Intelligence Tools:**
  * **Netcraft:** Discovers web server operating systems, web frameworks, hosting histories, SSL/TLS certificates, and infrastructure shifts.
  * **Web Spiders & Crawlers (Photon, HTTrack, Burp Spider):** Recursively parse target web trees to enumerate hidden endpoints, client-side JavaScript comments, and sensitive forms.
* **WHOIS Lookups:** Queries Regional Internet Registries (RIRs like ARIN, RIPE, APNIC) to retrieve domain ownership, administrative contacts, registration dates, and authoritative name server addresses.
* **DNS Record Types & Reference Table:**

| Record Type | Description & Purpose | Exam Hook / Significance |
| :--- | :--- | :--- |
| **A** | Maps Hostname to IPv4 Address | Direct host address resolution |
| **AAAA** | Maps Hostname to IPv6 Address | Identifies dual-stack / IPv6 infrastructure |
| **MX** | Mail Exchange Record | Points to incoming mail server gateways |
| **NS** | Name Server Record | Identifies authoritative DNS servers for a domain |
| **CNAME** | Canonical Name (Alias) | Links subdomains to external services (SaaS targets) |
| **TXT** | Text Metadata (SPF, DKIM, DMARC) | Essential for email authentication and domain ownership |
| **SOA** | Start of Authority | Identifies master DNS server, serial number, and zone timers |
| **PTR** | Pointer Record (Reverse DNS) | Resolves IP address to Hostname (Inverse lookup) |
| **SRV** | Service Location Record | Defines port and protocol for specific services (e.g., Active Directory) |

### Network & Infrastructure Reconnaissance
* **Network Path & Traceroute Analysis:**
  * **Linux `traceroute`:** Sends UDP (or ICMP) packets with incrementally increasing Time-To-Live (TTL) values starting at 1.
  * **Windows `tracert`:** Uses ICMP Echo Request packets by default.
  * **Mechanism:** When a router receives a packet with `TTL = 1`, it drops the packet and responds with an `ICMP Time Exceeded` message (Type 11, Code 0), revealing its IP address.
  * **CEH Firewalk Hook:** Firewalk uses TTL calculations to determine firewall port-filtering rules by analyzing responses from hosts behind a gateway.

---

## Tier 3: CLI, OSINT Tools & Frameworks

### Comprehensive OSINT Framework Mapping

| Tool / Framework | Primary Capability | Key Technical Feature / Command Line Usage |
| :--- | :--- | :--- |
| **Recon-ng** | Automated Python OSINT framework | Modular interface (`modules load`, `db insert`, `run`) using API integrations |
| **theHarvester** | OSINT Email & Subdomain Harvester | Combines search engines, SHODAN, and PGP keys (`theharvester -d target.com -b all`) |
| **Shodan** | Search Engine for Internet-Connected Devices | Filters by service, open ports, and banners (`net:192.168.1.0/24 org:"Target"`) |
| **Maltego** | Graphical Link Analysis & Intelligence | Visual node graphs mapping relationships between IPs, domains, and individuals |
| **Sherlock** | Social Media Handle Enumeration | Queries hundreds of web services for specific usernames (`sherlock targetuser`) |
| **OSFW (OSRFramework)** | Deep Web & User Account Profiling | Set of Python libraries for username checks and domain profiling (`us3rsearch`) |
| **Censys** | Infrastructure & Certificate Search | Analyzes SSL/TLS certificate transparency logs to find hidden subdomains |

### DNS Zone Transfer Execution Commands
* **Concept:** Misconfigured DNS servers allow unauthorized requests for entire zone databases (`AXFR`), revealing all internal subdomains and IP mappings.
* **CLI Execution:**
  * `dig axfr @ns1.target.com target.com`
  * `nslookup` -> `set type=any` -> `ls -d target.com`

---

## Tier 4: Real-World Scenarios & Countermeasures

### Defensive Mitigations & Best Practices
* **DNS Hardening:**
  * Restrict DNS Zone Transfers (`AXFR`) strictly to named secondary DNS servers via IP access control lists (ACLs) or TSIG (Transaction Signature) keys.
  * Implement **Split-Horizon DNS** (Split-Brain DNS) to isolate internal network record resolution from public-facing DNS.
* **Public Information & Data Leakage Control:**
  * Enforce **WHOIS Privacy Services** or substitute official corporate support details in registration records instead of specific employee PII.
  * Continually audit public code repositories (GitHub, GitLab) for inadvertently exposed hardcoded API keys, certificates, or database credentials using tools like TruffleHog or GitLeaks.
* **Administrative & Email Controls:**
  * Deploy `robots.txt` and `X-Robots-Tag` directives carefully (Note: `robots.txt` prevents public search engine indexing, but can be read by attackers to locate sensitive directories).
  * Configure strict email security controls including **SPF (Sender Policy Framework)**, **DKIM (DomainKeys Identified Mail)**, and **DMARC (Domain-based Message Authentication, Reporting, and Conformance)**.

---

## Tier 5: Exam Triggers & Master Summary Table

| Exam Scenario / Keyword | Target / Mechanism | CEH High-Yield Answer / Tool |
| :--- | :--- | :--- |
| **Silent Reconnaissance** | Gathering info without touching target systems | **Passive Footprinting** (Shodan, WHOIS, Social Media) |
| **Direct Host Interaction** | Sending packets directly to target IPs | **Active Footprinting** (Nmap ping sweep, DNS AXFR) |
| **Find Administrative Login Portals** | Search query for specific URL strings | `inurl:admin` / `inurl:login` |
| **Uncover Hidden PDFs/XLS Files** | Search query restricting file formats | `filetype:pdf` or `filetype:xls` |
| **Extract Full Internal Subdomains** | Querying misconfigured DNS server | **DNS Zone Transfer** (`dig axfr @ns1.target.com target.com`) |
| **Map Reverse IP to Hostname** | Resolving IP address back to domain | **PTR Record** (Pointer Record) |
| **Verify Email Authentication Policies** | Text metadata record for domain validation | **TXT Record** (SPF / DKIM / DMARC) |
| **Trace Network Hops & TTL** | Packet path mapping via TTL expiry | `traceroute` (Linux / UDP) / `tracert` (Windows / ICMP) |
| **Gather OSINT via API Integrations** | Python-based modular CLI framework | **Recon-ng** |
| **Harvest Emails & Domains via CLI** | Public engine scraping tool | `theharvester -d target.com -b all` |
| **Find Connected IoT/Server Banners** | Public search engine for active hardware | **Shodan** / **Censys** |
| **Isolate Public vs. Internal DNS** | DNS architecture protection | **Split-Horizon DNS** |
| **Prevent External DNS Dumps** | DNS administrative protection | **Disable AXFR Transfers** / Restrict via ACLs |