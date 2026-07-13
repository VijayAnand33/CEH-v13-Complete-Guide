# Module 13: Hacking Web Servers

> **Status:** ✅ Completed
>
> **Difficulty:** ⭐⭐⭐⭐☆
>
> **Labs Completed:** 3
>
> **Tools Covered:** Netcat, Telnet, Nmap, Hydra, Searchsploit, ShellGPT

---

# Module Summary

This module focuses on assessing the security of web servers through footprinting, service enumeration, password auditing, vulnerability exploitation, and AI-assisted security analysis. The practical exercises demonstrate how ethical hackers identify exposed services, enumerate web server configurations, perform password attacks against FTP services, exploit vulnerable web applications such as Log4Shell, and leverage AI tools to assist during web server penetration testing.

Through the practical labs, I learned how to gather web server information using Netcat, Telnet, and Nmap, audit FTP credentials with Hydra, exploit vulnerable applications using Metasploit, and utilize ShellGPT to assist with reconnaissance and attack planning.

---

# Overview

Web servers are critical components of modern organizations, hosting websites, web applications, APIs, and online services that are accessible over the Internet. Because they are publicly exposed, attackers frequently target web servers to exploit misconfigurations, weak authentication mechanisms, outdated software, and known vulnerabilities. A compromised web server can lead to unauthorized access, sensitive data exposure, website defacement, or complete system compromise.

This module demonstrates the methodology used to assess web server security through footprinting, service enumeration, password auditing, vulnerability exploitation, and AI-assisted reconnaissance. It also emphasizes secure server configuration, timely patch management, strong authentication practices, and continuous monitoring as essential defensive measures.

---

# Learning Objectives

After completing this module, I was able to:

- Understand the architecture and purpose of web servers.
- Perform web server footprinting using Netcat and Telnet.
- Enumerate web server information using Nmap NSE scripts.
- Identify exposed web services and server configurations.
- Perform FTP password auditing using Hydra.
- Exploit the Log4Shell (Log4j) vulnerability using Metasploit.
- Utilize ShellGPT to assist with web server reconnaissance and security analysis.
- Recommend best practices for securing web servers against common attacks.

---

# Key Concepts

- Web Server
- HTTP
- HTTPS
- Banner Grabbing
- Web Server Footprinting
- Web Server Enumeration
- Nmap NSE
- FTP Authentication
- Dictionary Attack
- Log4Shell (Log4j)
- Remote Code Execution (RCE)
- Web Server Hardening
- AI-Assisted Penetration Testing

---

# Tools Used

- [Netcat](../../Tools/Netcat.md)
- [Telnet](../../Tools/Telnet.md)
- [Nmap](../../Tools/Nmap.md)
- [Hydra](../../Tools/Hydra.md)
- [Searchsploit](../../Tools/Searchsploit.md)
- [ShellGPT](../../Tools/ShellGPT.md)

---

# Labs Covered

| Lab | Description |
|------|-------------|
| Lab 1 | Footprint the web server using Netcat, Telnet, and Nmap NSE. |
| Lab 2 | Perform a web server attack by cracking FTP credentials and exploiting the Log4j vulnerability. |
| Lab 3 | Perform AI-assisted web server footprinting and attack analysis using ShellGPT. |

---

# Lab 1 - Footprint the Web Server

## Objective

To perform web server footprinting using Netcat, Telnet, and the Nmap Scripting Engine (NSE) in order to identify server information, enumerate web application directories, discover hostnames, detect HTTP TRACE support, and identify the presence of a Web Application Firewall (WAF).

---

## Background

Web server footprinting is the process of gathering information about a target web server before attempting further security assessments. Information such as server banners, software versions, enabled HTTP methods, exposed directories, and security controls helps penetration testers identify potential vulnerabilities and understand the server's attack surface. Common footprinting techniques include banner grabbing, directory enumeration, hostname discovery, and security configuration analysis using specialized tools.

---

## Task 1 - Footprint a Web Server using Netcat and Telnet

### Tools Used

- [Netcat](../../Tools/Netcat.md)
- [Telnet](../../Tools/Telnet.md)

---

### Activity Performed

Netcat was used to establish a TCP connection to the target web server over port 80. A manual HTTP GET request was sent to perform banner grabbing and retrieve HTTP response headers, revealing information such as the server type, content type, modification date, ETag, and other server details.

The same banner-grabbing technique was then repeated using Telnet. By manually sending an HTTP request, the server's response headers were captured and analyzed. Both utilities demonstrated how simple TCP clients can be used to fingerprint web servers and gather valuable reconnaissance information before further security assessment.

---

### Observations

- Successfully connected to the target web server.
- Performed HTTP banner grabbing using Netcat.
- Retrieved server response headers and server information.
- Repeated banner grabbing using Telnet.
- Understood how banner information assists during web server reconnaissance.

---

### Banner Grabbing using Netcat

![Netcat Banner Grabbing](Images/Lab1-Netcat-Banner-Grabbing.png)

*Figure 1.1 – HTTP banner grabbing using Netcat to retrieve web server response headers and server information.*

---

### Banner Grabbing using Telnet

![Telnet Banner Grabbing](Images/Lab1-Telnet-Banner-Grabbing.png)

*Figure 1.2 – HTTP banner grabbing using Telnet to identify the target web server and its response headers.*

---

### Learning Outcome

This task demonstrated how Netcat and Telnet can be used to manually perform web server footprinting through banner grabbing. I learned how HTTP response headers reveal valuable server information that assists during reconnaissance and vulnerability assessment.

---

### Attack Flow

```mermaid
graph TD
    A["Target Web Server"] --> B["Connect using Netcat / Telnet"]
    B --> C["Send HTTP GET Request"]
    C --> D["Receive HTTP Response"]
    D --> E["Identify Server Information"]
```

---

## Task 2 - Enumerate Web Server Information using Nmap Scripting Engine (NSE)

### Tools Used

- [Nmap](../../Tools/Nmap.md)

---

### Activity Performed

Nmap NSE scripts were executed against the target web server to enumerate publicly accessible resources and identify potential security weaknesses. The `http-enum` script was used to discover web application directories, `hostmap-bfk` identified associated hostnames, `http-trace` verified whether the HTTP TRACE method was enabled, and `http-waf-detect` checked for the presence of a Web Application Firewall protecting the target server.

---

### Observations

- Enumerated web application directories using `http-enum`.
- Discovered associated hostnames using `hostmap-bfk`.
- Verified HTTP TRACE configuration.
- Checked for the presence of a Web Application Firewall.
- Learned how NSE scripts automate advanced web server enumeration.

---

### Web Directory Enumeration

![Nmap HTTP Enum](Images/Lab1-Nmap-HTTP-Enum.png)

*Figure 1.3 – Nmap NSE enumerating publicly accessible web application directories.*

---

### Hostname Enumeration

![Nmap Hostmap](Images/Lab1-Nmap-Hostmap.png)

*Figure 1.4 – Hostname discovery using the Nmap `hostmap-bfk` NSE script.*

---

### HTTP TRACE Detection

![Nmap HTTP Trace](Images/Lab1-Nmap-HTTP-Trace.png)

*Figure 1.5 – Detecting whether the HTTP TRACE method is enabled on the target web server.*

---

### Web Application Firewall Detection

![Nmap WAF Detection](Images/Lab1-Nmap-WAF-Detection.png)

*Figure 1.6 – Detecting the presence of a Web Application Firewall using the Nmap `http-waf-detect` NSE script.*

---

### Learning Outcome

This task demonstrated how the Nmap Scripting Engine automates web server enumeration by identifying exposed resources, discovering hostnames, testing HTTP methods, and detecting security controls. I learned how NSE scripts significantly improve reconnaissance efficiency during web server security assessments.

---

### Attack Flow

```mermaid
graph TD
    A["Target Web Server"] --> B["Run Nmap NSE Scripts"]
    B --> C["Enumerate Directories"]
    B --> D["Discover Hostnames"]
    B --> E["Check HTTP TRACE"]
    B --> F["Detect WAF"]
    C --> G["Analyze Results"]
    D --> G
    E --> G
    F --> G
```

---

## Overall Learning Outcome

This lab demonstrated how web server footprinting combines manual banner grabbing with automated enumeration techniques to gather valuable information about target servers. Using Netcat, Telnet, and Nmap NSE scripts, I learned how to identify server characteristics, enumerate exposed resources, evaluate security configurations, and build a strong reconnaissance foundation before conducting further web server security testing.

---

# Lab 2 - Perform a Web Server Attack

## Objective

To assess the security of a target web server by performing password auditing against an FTP service using a dictionary attack and exploiting the Log4Shell (Log4j) vulnerability to obtain remote command execution.

---

## Background

After completing reconnaissance and enumeration, penetration testers attempt to identify and exploit weaknesses present on exposed services. Weak passwords, vulnerable applications, and outdated software often provide attackers with opportunities to gain unauthorized access. Password auditing validates the strength of authentication mechanisms, while vulnerability exploitation demonstrates the impact of unpatched software. Understanding these attack techniques enables organizations to strengthen authentication policies and promptly remediate critical vulnerabilities.

---

## Task 1 - Crack FTP Credentials using a Dictionary Attack

### Tools Used

- [Nmap](../../Tools/Nmap.md)
- [Hydra](../../Tools/Hydra.md)

---

### Activity Performed

The target Windows 11 machine was first scanned using Nmap to verify that the FTP service was accessible over port 21. After confirming that the service was running, an initial login attempt with random credentials failed, indicating that authentication was required. A dictionary attack was then performed using Hydra with predefined username and password wordlists. Hydra successfully identified valid FTP credentials, which were subsequently used to authenticate to the FTP server. After gaining access, a new directory was created remotely to verify successful control of the FTP service.

---

### Observations

- Confirmed that FTP was running on port 21.
- Verified that authentication was required for FTP access.
- Successfully performed a dictionary attack using Hydra.
- Recovered valid FTP credentials.
- Authenticated to the FTP server using the recovered credentials.
- Successfully created a directory on the target system through FTP.

---

### FTP Port Enumeration

![Nmap FTP Port Scan](Images/Lab2-Nmap-FTP-Port-Scan.png)

*Figure 2.1 – Nmap identifying the FTP service running on port 21 of the target system.*

---

### Dictionary Attack using Hydra

![Hydra Dictionary Attack](Images/Lab2-Hydra-Dictionary-Attack.png)

*Figure 2.2 – Hydra performing a dictionary attack and successfully recovering valid FTP credentials.*

---

### Successful FTP Authentication

![FTP Login Success](Images/Lab2-FTP-Login-Success.png)

*Figure 2.3 – Successfully authenticating to the FTP server using the recovered credentials.*

---

### Remote Directory Creation

![FTP Remote Directory](Images/Lab2-FTP-Remote-Directory.png)

*Figure 2.4 – Verifying remote access by creating a new directory on the target FTP server.*

---

### Learning Outcome

This task demonstrated how weak passwords can be exploited using dictionary attacks. I learned how Nmap identifies exposed services and how Hydra automates password auditing to recover valid credentials, emphasizing the importance of strong password policies and secure authentication mechanisms.

---

### Attack Flow

```mermaid
graph TD
    A["Scan Target"] --> B["Identify FTP Service"]
    B --> C["Run Hydra Dictionary Attack"]
    C --> D["Recover Credentials"]
    D --> E["Authenticate to FTP Server"]
    E --> F["Gain Remote Access"]
```

---

## Task 2 - Gain Access to Target Web Server by Exploiting Log4j Vulnerability

### Tools Used

- [Nmap](../../Tools/Nmap.md)
- [Searchsploit](../../Tools/Searchsploit.md)
- [Netcat](../../Tools/Netcat.md)

---

### Activity Performed

A vulnerable Log4j server was deployed within the Ubuntu environment. Nmap was used to enumerate running services and identify Apache Tomcat on port 8080. Searchsploit was then used to identify the Log4Shell Remote Code Execution vulnerability affecting the target service. A proof-of-concept exploit generated a malicious payload, which was injected into the vulnerable web application through the login page. Upon processing the payload, the target server established a reverse shell connection back to the attacker's Netcat listener, providing remote command execution on the compromised system.

---

### Observations

- Successfully identified the vulnerable Apache Tomcat service.
- Confirmed the presence of a Log4Shell Remote Code Execution vulnerability.
- Generated a malicious payload using the proof-of-concept exploit.
- Delivered the payload through the vulnerable web application.
- Established a reverse shell connection.
- Verified remote command execution with root privileges.

---

### Vulnerable Log4j Server

![Log4j Vulnerable Server](Images/Lab2-Log4j-Vulnerable-Server.png)

*Figure 2.5 – Vulnerable Log4j application deployed and running on the target server.*

---

### Service Enumeration

![Nmap Service Enumeration](Images/Lab2-Nmap-Service-Enumeration.png)

*Figure 2.6 – Nmap identifying Apache Tomcat running on port 8080.*

---

### Vulnerability Identification

![Searchsploit Log4j](Images/Lab2-Searchsploit-Log4j.png)

*Figure 2.7 – Searchsploit identifying available Log4Shell Remote Code Execution exploits.*

---

### Payload Generation

![Log4j Payload Generated](Images/Lab2-Log4j-Payload-Generated.png)

*Figure 2.8 – Generating the Log4Shell exploit payload for remote code execution.*

---

### Payload Injection

![Log4j Payload Injection](Images/Lab2-Log4j-Payload-Injection.png)

*Figure 2.9 – Injecting the generated payload into the vulnerable web application.*

---

### Reverse Shell Established

![Reverse Shell Established](Images/Lab2-Reverse-Shell-Established.png)

*Figure 2.10 – Reverse shell connection successfully established on the attacker's machine.*

---

### Root Shell Verification

![Root Shell Verification](Images/Lab2-Root-Shell-Verification.png)

*Figure 2.11 – Verifying successful remote code execution by confirming root-level shell access.*

---

### Learning Outcome

This task demonstrated the complete exploitation workflow of a critical web application vulnerability. I learned how attackers identify vulnerable services, validate publicly known exploits, generate malicious payloads, and obtain remote command execution. The exercise reinforced the importance of timely patch management, vulnerability assessment, and secure application deployment.

---

### Attack Flow

```mermaid
graph TD
    A["Deploy Vulnerable Server"] --> B["Enumerate Services"]
    B --> C["Identify Log4Shell Vulnerability"]
    C --> D["Generate Exploit Payload"]
    D --> E["Inject Payload"]
    E --> F["Reverse Shell"]
    F --> G["Remote Code Execution"]
```

---

## Overall Learning Outcome

This lab demonstrated both authentication attacks and web application exploitation against a target web server. By performing password auditing with Hydra and exploiting the Log4Shell vulnerability, I gained practical experience in identifying exposed services, validating vulnerabilities, obtaining unauthorized access, and understanding the importance of strong authentication, secure software development, and timely security patching.

---

# Lab 3 - Perform Web Server Hacking using AI

## Objective

To utilize ShellGPT for automating web server footprinting, reconnaissance, and attack-related tasks using AI-generated security commands.

---

## Background

Artificial Intelligence has become an important component of modern penetration testing by assisting security professionals with reconnaissance, vulnerability assessment, and command generation. ShellGPT enables ethical hackers to convert natural language prompts into executable security commands, reducing manual effort while improving efficiency during web server security assessments.

---

## Task 1 - Perform Web Server Footprinting and Attacks using ShellGPT

### Tools Used

- [ShellGPT](../../Tools/ShellGPT.md)

---

### Activity Performed

ShellGPT was used to generate penetration testing commands through natural language prompts. AI-assisted command generation was utilized to perform web server footprinting, automate reconnaissance activities, and generate commands for website mirroring. The generated commands were executed to assess the target web server and perform security testing more efficiently than manually constructing each command.

---

### Observations

- Generated penetration testing commands using natural language prompts.
- Automated web server footprinting using AI.
- Generated website mirroring commands.
- Executed AI-generated commands successfully.
- Improved efficiency during web server reconnaissance and assessment.

---

### AI-Assisted Web Server Footprinting

![ShellGPT Web Footprinting](Images/Lab3-ShellGPT-Web-Footprinting.png)

*Figure 3.1 – ShellGPT generating commands to perform automated web server footprinting.*

---

### AI-Assisted Website Mirroring

![ShellGPT Website Mirroring](Images/Lab3-ShellGPT-Website-Mirroring.png)

*Figure 3.2 – ShellGPT generating commands to mirror the target website for offline analysis.*

---

### Mirrored Website

![Mirrored Website](Images/Lab3-Mirrored-Website.png)

*Figure 3.3 – Successfully mirrored website opened locally after executing the AI-generated command.*

---

### Learning Outcome

This task demonstrated how AI-powered assistants can simplify penetration testing by automatically generating security commands for reconnaissance and web server assessment. I learned how ShellGPT accelerates web server testing by reducing manual command creation while maintaining an efficient penetration testing workflow.

---

### Attack Flow

```mermaid
graph TD
    A["Natural Language Prompt"] --> B["ShellGPT"]
    B --> C["Generate Security Commands"]
    C --> D["Execute Commands"]
    D --> E["Footprint Web Server"]
    D --> F["Mirror Website"]
    E --> G["Analyze Results"]
    F --> G
```

---

## Overall Learning Outcome

This lab demonstrated the practical use of AI to support web server security assessments. Using ShellGPT, I learned how natural language prompts can generate penetration testing commands for reconnaissance, footprinting, and website analysis, improving both the speed and efficiency of ethical hacking workflows.

---

# Key Takeaways

- Understood the methodology used to perform web server footprinting and reconnaissance.
- Performed banner grabbing using Netcat and Telnet to identify web server information.
- Enumerated web server resources, hostnames, HTTP methods, and Web Application Firewall (WAF) configurations using Nmap NSE scripts.
- Performed FTP password auditing using Hydra and successfully gained remote access through a dictionary attack.
- Exploited the Log4Shell (Log4j) vulnerability to obtain remote command execution on a vulnerable web server.
- Utilized Searchsploit to identify publicly available exploits targeting vulnerable services.
- Explored how AI-assisted tools such as ShellGPT can automate web server reconnaissance and command generation.
- Reinforced the importance of secure authentication, vulnerability management, timely patching, and continuous web server monitoring.

---

# Defensive Perspective

Web servers are among the most frequently targeted assets within an organization's infrastructure. Protecting them requires a layered security approach that includes secure configuration, regular vulnerability assessments, strong authentication policies, continuous patch management, and web application firewalls. Organizations should disable unnecessary services, enforce complex password policies, monitor authentication attempts, regularly scan for vulnerabilities, and promptly remediate critical issues such as Log4Shell. AI-assisted security tools can further improve defensive operations by accelerating reconnaissance, automation, and security analysis.

---

# Interview Questions

1. What information can be obtained through web server footprinting?
2. How does banner grabbing assist during reconnaissance?
3. What are Nmap NSE scripts and why are they useful?
4. How does Hydra perform a dictionary attack?
5. Why are weak passwords a significant security risk?
6. What is the Log4Shell (Log4j) vulnerability?
7. How does a reverse shell differ from a normal shell?
8. What role does Searchsploit play during penetration testing?
9. How can organizations protect web servers against common attacks?
10. How can AI tools such as ShellGPT assist ethical hackers during security assessments?

---

# My Reflection

This module provided practical experience in assessing web server security through reconnaissance, enumeration, password auditing, vulnerability exploitation, and AI-assisted penetration testing. Performing banner grabbing, exploiting weak authentication, and demonstrating the impact of the Log4Shell vulnerability strengthened my understanding of both offensive techniques and defensive best practices. The exercises also highlighted how AI can improve efficiency by automating command generation while maintaining a structured penetration testing workflow.

---
