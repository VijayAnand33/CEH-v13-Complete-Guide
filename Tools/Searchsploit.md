# Searchsploit

## Overview

Searchsploit is a command-line utility that allows security professionals to search the local Exploit Database (Exploit-DB) for publicly available exploits. It enables penetration testers to quickly identify exploits related to discovered software versions and known vulnerabilities without requiring an Internet connection.

---

## Key Features

- Searches the local Exploit Database for known vulnerabilities.
- Identifies exploits based on software name or version.
- Supports offline exploit searching.
- Filters search results using keywords.
- Displays exploit paths stored on the local system.
- Integrates well with reconnaissance and vulnerability assessment workflows.

---

## Common Usage

```bash
searchsploit apache

searchsploit tomcat

searchsploit -t Apache RCE

searchsploit log4j

searchsploit --help
```

---

## Practical Example

During **Module 13 – Hacking Web Servers**, Searchsploit was used after service enumeration with Nmap to identify publicly available Remote Code Execution (RCE) exploits affecting the detected Apache Tomcat service. The identified Log4Shell (Log4j) vulnerability was then selected for exploitation during the lab.

---

## Defensive Perspective

Security professionals use Searchsploit during vulnerability assessments to verify whether discovered software versions have publicly available exploits. This helps organizations prioritize patching, validate vulnerability scan results, and understand the potential impact of exposed services before attackers can exploit them.

---

## Used In

- Module 13: Hacking Web Servers

---

## Official Resources

- https://www.exploit-db.com/searchsploit
- https://www.exploit-db.com

---