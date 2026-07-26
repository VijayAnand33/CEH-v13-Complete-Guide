# Module 02: Footprinting and Reconnaissance

## 2.1 Footprinting Concepts

### Principles of Footprinting
* **Footprinting Definition:** The systematic first step of ethical hacking where an attacker or tester gathers comprehensive information about a target organization, network, or system.
* **Objectives:**
  * Discover network architecture, open ranges, domain names, and administrative contacts.
  * Identify specific vulnerabilities, misconfigurations, and entry points for active exploitation.
  * Construct a clear threat landscape without triggering security alerts during passive phases.
* **Passive vs. Active Footprinting:**
  * **Passive Footprinting:** Gathering target intelligence without direct interaction with the target systems (e.g., searching OSINT, public records, social media, WHOIS databases).
  * **Active Footprinting:** Direct communication with target systems to harvest technical operational details (e.g., executing network traceroutes, DNS queries, web crawling).

---

## 2.2 Footprinting Methodology

### Search Engine Reconnaissance
* **Advanced Google Search Operators (Google Dorks):**
  * `site:` Restricts results to a specific domain or host (e.g., `site:target.com`).
  * `filetype:` Limits results to specific document formats (e.g., `filetype:pdf`, `filetype:xls`).
  * `inurl:` Filters for URLs containing specific strings (e.g., `inurl:admin`, `inurl:login`).
  * `intitle:` Restricts search to pages containing specific keywords in the HTML title tag.
* **Google Hacking Database (GHDB):** A public collection of search queries designed to uncover exposed credentials, vulnerable scripts, server advisories, and administrative interface pages.

### Web & Domain Reconnaissance
* **Website Intelligence Gathering:**
  * **Netcraft:** Used for discovering target web server technologies, operating system signatures, hosting histories, and associated subdomains.
  * **Web Spiders & Crawlers:** Automated tools (e.g., Photon, HTTrack) used to recursively parse site structure, hidden directories, forms, and JavaScript parameters.
* **WHOIS & Domain Registries:**
  * Identifies domain registration details including owner contacts, domain registrar, creation/expiry dates, and authoritative name servers.
* **DNS Footprinting & Record Enumeration:**
  * **A Record:** Maps a hostname to an IPv4 address.
  * **AAAA Record:** Maps a hostname to an IPv6 address.
  * **MX Record:** Specifies mail servers responsible for receiving domain email.
  * **NS Record:** Identifies authoritative name servers managing the domain's DNS zones.
  * **CNAME Record:** Specifies alias names pointing to canonical hostnames.
  * **TXT Record:** Holds text data, often utilized for SPF (Sender Policy Framework), DKIM, and domain verification.
  * **DNS Zone Transfer:** Attempting to retrieve a complete DNS zone database from misconfigured authoritative name servers using tools like `dig` or `nslookup` (e.g., `dig axfr @ns1.target.com target.com`).

### Network & Infrastructure Reconnaissance
* **Network Path & Route Analysis:**
  * **Traceroute (Unix/Linux) / Tracert (Windows):** Identifies the network hop-by-hop path taken by packets toward a destination host by analyzing ICMP/UDP Time-To-Live (TTL) expiration responses.
  * Uncovers topology, firewall locations, perimeter routers, and internal subnet boundaries.
* **IP Address Geolocation:** Mapping target public IP subnets to geographic coordinates, ISP backbones, and host facilities.

### Social Engineering & Person Reconnaissance
* **People Search Engines & OSINT:** Utilizing platforms like Pipl, Spokeo, and social networks (LinkedIn, Twitter) to gather employee directories, corporate structures, email formats, and tech stacks mentioned in job postings.
* **Username & Account Enumeration:** Running tools like Sherlock to search hundreds of social networks and web platforms for specific target handles and profiles.

---

## 2.3 Footprinting Tools & Frameworks

### OSINT & Reconnaissance Frameworks
* **Recon-ng:** A modular, web-based open-source intelligence framework written in Python used to perform automated domain, contact, host, and vulnerability gathering via various API integrations.
* **ShellGPT / AI-Driven Automation:** Utilizing local or automated scripting interfaces (e.g., ShellGPT) to automate command execution for email harvesting, subdomain discovering, and target report generation.
* **Email Tracking Tools:**
  * **eMailTrackerPro:** Analyzes email headers to determine origin IP addresses, intermediate proxy relays, and geolocation information of the sender.

---

## 2.4 Footprinting Countermeasures

### Security Defenses & Prevention
* **DNS Protection:**
  * Disable unauthorized DNS zone transfers (`AXFR`) to external or unauthenticated IP addresses.
  * Use split-horizon DNS architecture to separate internal domain infrastructure from external public-facing DNS.
* **OSINT & Public Exposure Mitigation:**
  * Enforce WHOIS privacy services or substitute corporate information instead of employee PII in registration fields.
  * Audit corporate websites and public code repositories (e.g., GitHub) to prevent leaks of API keys, configuration credentials, or detailed internal infrastructure diagrams.
* **Technical Controls:**
  * Restrict administrative interfaces from being indexed by search engines via appropriate directives (e.g., `robots.txt`, `X-Robots-Tag`).
  * Implement strict email authentication frameworks (SPF, DKIM, DMARC) to prevent email spoofing and domain misuse.