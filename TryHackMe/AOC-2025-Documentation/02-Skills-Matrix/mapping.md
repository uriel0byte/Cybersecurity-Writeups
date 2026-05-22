# Technical Skills Matrix — Advent of Cyber 2025

**Completed:** December 1–24, 2025 | **Platform:** TryHackMe | **Days:** 24/24  
**Evidence:** Full writeups with screenshots, SPL/KQL queries, tool workflows, and room answers documented in `/03-Daily-Challenges/`.

This matrix maps hands-on lab experience from the 24-day structured challenge series to SOC analyst skill requirements and Security+ SY0-701 exam domains. Proficiency ratings reflect actual lab performance in guided environments — not independent real-world experience.

---

## Skills Overview

| Skill Area | Days | Key Tools | Proficiency | SOC Analyst Application |
|---|---|---|---|---|
| SIEM & Log Analysis | 3, 10, 15, 22 | Splunk (SPL), Microsoft Sentinel (KQL), RITA, Zeek | ⭐⭐⭐☆☆ | Alert triage, log correlation, multi-source investigation, attack timeline reconstruction |
| Web Attack Forensics | 3, 15 | Splunk, Apache access/error logs, Sysmon | ⭐⭐⭐☆☆ | Command injection detection, parent-child process analysis, IOC extraction |
| Malware Analysis | 6, 13, 21 | PeStudio, ProcMon, Regshot, YARA, CyberChef, pluma | ⭐⭐⭐☆☆ | Static/dynamic analysis, IOC identification, persistence detection, HTA analysis |
| Email Security & Phishing | 2, 12 | SET, email header analysis, SPF/DKIM/DMARC | ⭐⭐⭐☆☆ | Phishing detection, spoofing identification, header analysis, punycode recognition |
| Linux Administration | 1, 7, 9, 22 | bash, grep, find, Nmap, Netcat, dig, ss | ⭐⭐⭐☆☆ | CLI investigation, log analysis, bash history forensics, service enumeration |
| Password Security | 9, 17 | John the Ripper, pdfcrack, zip2john, CrackStation | ⭐⭐⭐☆☆ | Offline cracking detection, hash identification, credential compromise response |
| Encoding & Obfuscation | 17, 18 | CyberChef, browser DevTools, PowerShell analysis | ⭐⭐⭐☆☆ | Payload deobfuscation, Base64/XOR/ROT decoding, malware evasion technique recognition |
| Network Security | 7 | Nmap, Netcat, dig, FTP, ss | ⭐⭐⭐☆☆ | Port scanning, service enumeration, protocol analysis, localhost vs. external service exposure |
| Windows Forensics | 6, 16 | Registry Explorer, Registry Editor, ProcMon, Regshot | ⭐⭐☆☆☆ | Registry hive analysis, persistence mechanism identification, artifact recovery |
| Web Application Security | 5, 11, 20, 24 | Browser DevTools, Burp Suite, cURL, bash | ⭐⭐☆☆☆ | IDOR, XSS, race conditions, HTTP manipulation, brute-force pattern recognition |
| C2 Detection & Threat Hunting | 22 | RITA, Zeek, PCAP analysis | ⭐⭐☆☆☆ | Beacon detection, behavioral analysis, metadata-driven hunting in encrypted traffic |
| Cloud Security — Azure | 10 | Microsoft Sentinel, KQL, Azure Portal | ⭐⭐☆☆☆ | Cloud SIEM operations, alert triage, Linux privilege escalation investigation |
| Cloud Security — AWS | 23 | AWS CLI, IAM, STS, S3 | ⭐⭐☆☆☆ | IAM enumeration, privilege escalation via role assumption, CloudTrail monitoring |
| Container Security | 14 | Docker, Docker CLI, docker exec | ⭐⭐☆☆☆ | Docker socket risk identification, container escape vectors, runtime hardening |
| ICS / OT Security | 19 | Python, pymodbus, Modbus TCP | ⭐⭐☆☆☆ | Industrial protocol basics, Modbus register/coil interaction, OT incident response |
| AI Security | 4, 8 | AI agents, prompt injection, ReAct/CoT | ⭐⭐☆☆☆ | Prompt injection recognition, AI-assisted SOC workflows, agentic AI attack surface |

**Proficiency Scale:**  
⭐☆☆☆☆ Exposure only &nbsp;|&nbsp; ⭐⭐☆☆☆ Guided lab experience &nbsp;|&nbsp; ⭐⭐⭐☆☆ Independent with reference &nbsp;|&nbsp; ⭐⭐⭐⭐☆ Confident, can troubleshoot &nbsp;|&nbsp; ⭐⭐⭐⭐⭐ Expert

---

## Security+ SY0-701 Domain Mapping

### Domain 1.0 — General Security Concepts (12%)

| Day | Topic | Alignment |
|---|---|---|
| 1 | Linux CLI | File permissions, access control, least privilege |
| 2 | Phishing | Social engineering, security awareness |
| 5 | IDOR | Authentication vs. authorization, session management |
| 8 | Prompt Injection | Emerging AI threats, application security |
| 9 | Password Cracking | Cryptography concepts, password policy |
| 11 | XSS | Secure coding, browser security model |
| 17 | CyberChef | Encoding vs. encryption, hash functions |
| 18 | Obfuscation | Encoding vs. encryption vs. obfuscation |

### Domain 2.0 — Threats, Vulnerabilities & Mitigations (22%)

| Day | Topic | Alignment |
|---|---|---|
| 2 | Phishing | Phishing vectors, credential harvesting, social engineering |
| 5 | IDOR | Broken access control (OWASP #1), privilege escalation |
| 6 | Malware Analysis | Static/dynamic analysis, persistence, IOC identification |
| 7 | Network Discovery | Reconnaissance, scanning, attack surface enumeration |
| 9 | Password Cracking | Brute-force attacks, offline cracking, credential attacks |
| 11 | XSS | Injection attacks, client-side vulnerabilities |
| 12 | Phishing Analysis | Threat intelligence, email spoofing, typosquatting, punycode |
| 13 | YARA Rules | Malware indicators, signature-based detection, IOCs |
| 14 | Container Security | Container escape, Docker socket misconfigurations |
| 15 | Web Forensics | Command injection, web attack vectors, exploitation |
| 17 | CyberChef | Obfuscation, encoding techniques |
| 18 | Obfuscation | Malware evasion, deobfuscation methodology |
| 19 | ICS/Modbus | OT/ICS vulnerabilities, unauthenticated protocol access |
| 20 | Race Conditions | TOCTOU, atomicity violations, application logic flaws |
| 21 | HTA Malware | Living off the Land, fileless malware, VBScript/PowerShell |
| 22 | C2 Detection | Command and control infrastructure, beaconing, DNS tunneling |
| 23 | AWS Security | Cloud privilege escalation, IAM misconfigurations |
| 24 | cURL Exploitation | Web API exploitation, HTTP attack patterns |

### Domain 3.0 — Security Architecture (18%)

| Day | Topic | Alignment |
|---|---|---|
| 1 | Linux CLI | OS security, file system architecture |
| 7 | Network Discovery | Network architecture, service placement, 0.0.0.0 vs 127.0.0.1 |
| 10 | Alert Triage | Cloud SIEM architecture (Azure Sentinel) |
| 14 | Container Security | Container isolation, namespaces, cgroups, microservices |
| 16 | Registry Forensics | Windows OS architecture, registry structure |
| 19 | ICS/Modbus | IT/OT network segmentation, industrial protocol design |
| 22 | C2 Detection | Network security monitoring, SPAN ports, NSM architecture |
| 23 | AWS Security | Cloud IAM architecture, least privilege, identity management |

### Domain 4.0 — Security Operations (28%)

| Day | Topic | Alignment |
|---|---|---|
| 3 | Splunk SIEM | SIEM operations, SPL queries, log analysis, alert triage |
| 4 | AI in Security | Security automation, AI-assisted SOC workflows |
| 6 | Malware Analysis | Incident response, evidence collection, sandbox analysis |
| 7 | Network Discovery | Service discovery, network monitoring, threat hunting |
| 9 | Password Cracking | Incident detection, offline cracking monitoring |
| 10 | SOC Alert Triage | Alert triage, KQL queries, incident investigation, escalation |
| 12 | Phishing Analysis | Email security operations, threat detection |
| 13 | YARA Rules | Threat hunting, custom rule creation, IOC-based scanning |
| 15 | Web Forensics | Incident response, multi-source log correlation, forensic analysis |
| 16 | Registry Forensics | Digital forensics, artifact recovery, timeline reconstruction |
| 21 | HTA Malware | Static malware analysis, IOC extraction, detection engineering |
| 22 | C2 Detection | Proactive threat hunting, behavioral analysis, RITA/Zeek |

### Domain 5.0 — Security Program Management (20%)
Not a focus of the Advent of Cyber format. No meaningful coverage.

---

## Tool Reference

### SIEM Platforms

| Tool | Days | What Was Done |
|---|---|---|
| Splunk (SPL) | 3, 15 | Multi-source log correlation, attack chain reconstruction, 7-phase ransomware investigation, web forensics across Apache and Sysmon indexes |
| Microsoft Sentinel (KQL) | 10 | 8-incident triage, Linux privilege escalation investigation across 4 hosts, MITRE ATT&CK mapping |
| RITA | 22 | C2 beacon detection, Zeek log import, threat modifier analysis, behavioral hunting against `malhare.net` |
| Zeek | 22 | PCAP-to-log conversion, conn/http/dns/ssl log analysis as RITA input |

### Malware Analysis

| Tool | Days | What Was Done |
|---|---|---|
| PeStudio | 6 | SHA256 extraction, string analysis, suspicious imports on HopHelper.exe |
| Process Monitor | 6 | TCP activity filtering, registry write detection, C2 confirmation |
| Regshot | 6 | Before/after registry comparison, Run key persistence confirmed |
| YARA | 13 | Rule written from scratch, regex pattern matching, recursive scan |
| pluma / text editor | 21 | Static HTA analysis, VBScript execution flow tracing |

### Network & Recon

| Tool | Days | What Was Done |
|---|---|---|
| Nmap | 7 | Full TCP scan (`-p-`), UDP scan (`-sU`), banner grabbing (`--script=banner`) |
| Netcat | 7 | Probed unknown service on port 25251, retrieved key fragment |
| dig | 7 | DNS TXT record query against non-standard DNS server |
| ss | 7 | On-host service discovery, found MySQL on localhost invisible to Nmap |

### Forensics

| Tool | Days | What Was Done |
|---|---|---|
| Registry Explorer | 16 | Loaded offline hives with SHIFT+Open (transaction log replay), found DroneManager Updater in Uninstall key, confirmed persistence via Run key |
| Sysmon | 15 | Parent-child process analysis (`ParentImage`), whoami recon detection, encoded PowerShell hunting |
| CyberChef | 17, 18, 21 | Multi-step decode recipes, Base64→ROT13, XOR with key, Lock 1–5 decode chains |

### Cloud & Infrastructure

| Tool | Days | What Was Done |
|---|---|---|
| AWS CLI | 23 | IAM enumeration, role assumption, S3 access, temporary credential export |
| Docker | 14 | Container listing, exec into monitoring container, socket exploitation, privilege escalation to deployer |
| pymodbus | 19 | Read holding registers and coils, wrote remediation sequence against live PLC |

---

## What This Series Covers and What It Doesn't

**Demonstrated through labs:**
- Following structured SOC investigation workflows from detection to documentation
- Using SIEM platforms (Splunk, Sentinel) to correlate logs and reconstruct attack chains
- Performing static and dynamic malware analysis using standard tools
- Reading and writing Splunk SPL and Microsoft Sentinel KQL queries with reference material
- Identifying phishing indicators including header analysis, SPF/DKIM/DMARC failures, punycode, and typosquatting
- Decoding multi-layer obfuscated payloads using CyberChef
- Understanding Windows registry forensics and persistence mechanisms
- Basic AWS IAM enumeration and understanding of privilege escalation via role assumption
- Introductory ICS/Modbus protocol interaction and remediation

**Not yet covered — next development priorities:**
- Incident response procedures beyond investigation: containment, eradication, recovery
- Independent query construction without templates (the consistent gap across SIEM work)
- Advanced threat hunting: hypothesis-driven, without guided room structure
- Python scripting for automation beyond guided modification
- Active Directory and Windows Event Log hunting in depth
- MITRE ATT&CK framework application to original investigations

## MITRE ATT&CK Mapping

Techniques observed or investigated across the 24-day series. Evidence column references the specific day and what was seen.

| Tactic | Technique ID | Technique Name | Day(s) | Evidence |
|---|---|---|---|---|
| Initial Access | T1566.001 | Phishing: Spearphishing Attachment | 2, 12, 21 | HTA file disguised as salary survey; HopHelper.exe delivered via email |
| Initial Access | T1190 | Exploit Public-Facing Application | 3, 15 | SQL injection against web server; command injection via CGI script |
| Execution | T1059.001 | Command and Scripting Interpreter: PowerShell | 15, 21 | `powershell.exe -nop -w hidden -c` launched from httpd.exe; fileless `$U→$C→$B→Invoke-Command` chain in HTA payload |
| Execution | T1218.005 | Signed Binary Proxy Execution: Mshta | 21 | `mshta.exe` executed `survey.hta` outside browser sandbox — no custom malware binary |
| Execution | T1204.002 | User Execution: Malicious File | 6, 21 | User ran `HopHelper.exe` (claimed to be scheduling tool); opened `survey.hta` (claimed to be salary survey) |
| Persistence | T1547.001 | Boot or Logon Autostart Execution: Registry Run Keys | 6, 16 | HopHelper.exe added to `HKCU\...\Run`; DroneManager `dronehelper.exe --background` added to Run key |
| Persistence | T1547.006 | Boot or Logon Autostart Execution: Kernel Modules | 10 | `malicious_mod.ko` inserted via `insmod` on compromised Linux hosts — survives reboots |
| Privilege Escalation | T1548.003 | Abuse Elevation Control Mechanism: Sudo | 10 | Alice added to sudoers group; `NOPASSWD:ALL` entry written to `/etc/sudoers` |
| Privilege Escalation | T1611 | Escape to Host | 14 | Docker socket mounted in monitoring container — used `docker exec` to pivot to privileged deployer container |
| Privilege Escalation | T1078.004 | Valid Accounts: Cloud Accounts | 23 | `sts:AssumeRole` used to escalate from limited `sir.carrotbane` user to `bucketmaster` role with S3 access |
| Defense Evasion | T1027 | Obfuscated Files or Information | 17, 18, 21 | Base64+XOR+ROT13 layered payload in SantaStealer.ps1; two-layer Base64→ROT13 in HTA second stage |
| Defense Evasion | T1036 | Masquerading | 6, 21 | HopHelper.exe presented as scheduling program; DroneManager Updater using legitimate software naming convention |
| Discovery | T1046 | Network Service Discovery | 7 | Nmap full TCP scan (`-p-`) and UDP scan against tbfc-devqa01 |
| Discovery | T1033 | System Owner/User Discovery | 15 | `whoami` executed via cmd.exe immediately after command injection confirmed code execution |
| Discovery | T1069.003 | Permission Groups Discovery: Cloud Groups | 23 | `aws iam list-roles` enumerated available roles; BucketMasterPolicy read before assuming role |
| Credential Access | T1110.002 | Brute Force: Password Cracking | 9 | Dictionary attack against encrypted PDF and ZIP using `pdfcrack` and `john` with `rockyou.txt` |
| Credential Access | T1003.002 | OS Credential Dumping: Security Account Manager | 10 | `cp /etc/shadow` executed on compromised hosts — shadow file staged for credential harvest |
| Collection | T1005 | Data from Local System | 1, 21 | `eggstrike.sh` collected wishlist data to `/tmp/dump.txt`; HTA collected `ComputerName` and `UserName` via `WScript.Network` |
| Command and Control | T1071.001 | Application Layer Protocol: Web Protocols | 6, 22 | HopHelper.exe used HTTP to C2 at `138.62.51.186`; rabbithole.malhare.net beacon detected over HTTPS |
| Command and Control | T1132.001 | Data Encoding: Standard Encoding | 15, 21 | Base64-encoded PowerShell in HTTP query parameter; Base64→ROT13 in downloaded C2 payload |
| Exfiltration | T1048 | Exfiltration Over Alternative Protocol | 3 | `backup.zip` and `logs.tar.gz` downloaded via curl/zgrab; 126KB transferred to attacker IP |
| Exfiltration | T1041 | Exfiltration Over C2 Channel | 21 | `ComputerName` and `UserName` sent via HTTP GET to `survey.bestfestiivalcompany.com/details` |
| Impact | T1486 | Data Encrypted for Impact | 3 | `bunnylock.bin` ransomware executed via webshell — deployed after data exfiltration |
| Impact | T0831 | Manipulation of Control (ICS) | 19 | HR0 register changed from 0 (Christmas) to 1 (Eggs) via unauthenticated Modbus TCP write |

---

## Interview Reference

**Resume bullet:**
```
Hands-on SOC analyst lab experience across 24 structured security challenges (TryHackMe
Advent of Cyber 2025): SIEM investigation using Splunk and Microsoft Sentinel, malware
analysis (static/dynamic), Windows Registry forensics, C2 detection with RITA/Zeek,
phishing analysis, YARA rule creation, and introductory cloud security (AWS IAM, Docker).
Full writeup documentation at: github.com/uriel0byte/Cybersecurity-Writeups
```

**STAR examples for interview:**

*SIEM Investigation (Days 3, 15)*
> Situation: Ransomware attack on TBFC web server / command injection against drone web UI.
> Task: Investigate using Splunk to identify attacker, trace attack chain, and extract IOCs.
> Action: Wrote SPL queries across web traffic, firewall, Apache, and Sysmon indexes; correlated seven attack phases; decoded Base64-encoded PowerShell payload.
> Result: Reconstructed complete attack timeline, identified C2 communication, confirmed 126KB of exfiltrated data.

*Malware Analysis (Days 6, 21)*
> Situation: Suspicious executable and malicious HTA file received via phishing email.
> Task: Investigate without executing on a production system.
> Action: Static analysis with PeStudio and pluma; dynamic analysis with Regshot and ProcMon; decoded two-layer payload (Base64→ROT13) in CyberChef.
> Result: Confirmed HTTP C2 to 138.62.51.186, registry Run key persistence, and fileless PowerShell execution chain.

*Threat Hunting (Day 22)*
> Situation: Suspicious quiet period after series of attacks on TBFC infrastructure.
> Task: Proactively hunt for C2 beaconing in captured network traffic.
> Action: Converted PCAP to Zeek logs, imported into RITA, applied beacon score and prevalence filters to isolate `rabbithole.malhare.net` as C2 infrastructure.
> Result: Identified active C2 beaconing through behavioral analysis without relying on known signatures.

---

**Last Updated:** January 2026  
**Days Completed:** 24/24  
**Security+ Domains Covered:** 1.0, 2.0, 3.0, 4.0 (strong); 5.0 (none)
