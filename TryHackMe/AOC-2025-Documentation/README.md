# Advent of Cyber 2025
*uriel0byte — TryHackMe*

All 24 days completed. December 1–24, 2025.

This is my full run through TryHackMe's Advent of Cyber 2025. I came in as an ILS
student mid-career pivot into cybersecurity, with no prior SOC experience. I used
this as a structured 24-day introduction to core Blue Team operations. The write-ups
are built for review and portfolio use, not just completion records.

![AOC 2025 Certificate](./09-Certificates/aoc-2025-certificate.png)

**At a glance**
- 24/24 days completed
- 50+ tools across 16 security domains
- Full write-ups with screenshots, verified room answers, and key takeaways
- MITRE ATT&CK mapping in `/02-Skills-Matrix/mapping.md`

---

## Repository Structure

```text
/AOC-2025-Documentation
├── README.md
├── /02-Skills-Matrix
│   └── mapping.md                    # Skills inventory, Security+ mapping, MITRE ATT&CK
├── /03-Daily-Challenges              # Core portfolio content
│   ├── day-01-linux-cli.md
│   ├── day-02-phishing.md
│   ├── day-03-splunk-siem.md
│   ├── day-04-ai-security.md
│   ├── day-05-idor.md
│   ├── day-06-malware-analysis.md
│   ├── day-07-network-discovery.md
│   ├── day-08-prompt-injection.md
│   ├── day-09-password-cracking.md
│   ├── day-10-soc-alert-triage.md
│   ├── day-11-xss.md
│   ├── day-12-phishing-analysis.md
│   ├── day-13-yara-rules.md
│   ├── day-14-container-security.md
│   ├── day-15-web-attack-forensics.md
│   ├── day-16-registry-forensics.md
│   ├── day-17-cyberchef.md
│   ├── day-18-obfuscation.md
│   ├── day-19-ics-modbus.md
│   ├── day-20-race-conditions.md
│   ├── day-21-hta-malware.md
│   ├── day-22-c2-detection-rita.md
│   ├── day-23-aws-security.md
│   └── day-24-curl-exploitation.md
├── /04-Case-Studies
│   └── case-study.md                 # Blue Team investigation showcase
├── /07-Screenshots
└── /09-Certificates
    └── aoc-2025-certificate.pdf
```

---

## Challenge Overview

| Day | Title | Category | Difficulty | Key Tools |
|---|---|---|---|---|
| 1 | Shells and Bells | Linux CLI | ★☆☆☆ | bash, grep, find |
| 2 | Merry Clickmas | Phishing | ★★☆☆ | SET, SMTP |
| 3 | Did you SIEM? | Splunk SIEM | ★★★☆ | Splunk, SPL |
| 4 | old sAInt nick | AI Security | ★☆☆☆ | AI agents |
| 5 | Santa's Little IDOR | IDOR | ★★★☆ | DevTools, Burp |
| 6 | Egg-xecutable | Malware Analysis | ★★★☆ | PeStudio, ProcMon |
| 7 | Scan-ta Clause | Network Discovery | ★★☆☆ | Nmap, Netcat |
| 8 | Sched-yule Conflict | Prompt Injection | ★★☆☆ | Agentic AI |
| 9 | A Cracking Christmas | Password Cracking | ★☆☆☆ | John, pdfcrack |
| 10 | Tinsel Triage | SOC Alert Triage | ★★★★ | Sentinel, KQL |
| 11 | Merry XSSMas | XSS | ★★☆☆ | Browser DevTools |
| 12 | Phishmas Greetings | Phishing Analysis | ★★★☆ | Email headers |
| 13 | YARA Mean One! | YARA Rules | ★★★☆ | YARA |
| 14 | DoorDasher's Demise | Container Security | ★★★☆ | Docker |
| 15 | Drone Alone | Web Forensics | ★★★★ | Splunk, Sysmon |
| 16 | Registry Furensics | Windows Forensics | ★★★☆ | Registry Explorer |
| 17 | Hoperation Save McSkidy | CyberChef | ★★★☆ | CyberChef |
| 18 | The Egg Shell File | Obfuscation | ★★★☆ | CyberChef, PowerShell |
| 19 | Claus for Concern | ICS/SCADA | ★★★★ | Python, pymodbus |
| 20 | Toy to The World | Race Conditions | ★★★☆ | Burp Suite |
| 21 | Malhare.exe | HTA Malware | ★★★★ | pluma, CyberChef |
| 22 | Command & Carol | C2 Detection | ★★★☆ | RITA, Zeek |
| 23 | S3cret Santa | AWS Security | ★★★☆ | AWS CLI, IAM |
| 24 | Hoperation Eggsploit | cURL Exploitation | ★★★☆ | cURL, bash |

★☆☆☆ Easy &nbsp;|&nbsp; ★★☆☆ Easy-Medium &nbsp;|&nbsp; ★★★☆ Medium &nbsp;|&nbsp; ★★★★ Hard (personal rating)

---

## Featured Write-ups

**[Day 3 — Splunk SIEM Log Analysis](./03-Daily-Challenges/day-03-splunk-siem.md)**

Traced a full ransomware attack chain using Splunk — from initial reconnaissance
through SQL injection, webshell deployment, and data exfiltration. Identified C2
communication, confirmed 126KB of stolen data, and pivoted between web traffic
and firewall log sources to close the picture.

---

**[Day 6 — Malware Analysis](./03-Daily-Challenges/day-06-malware-analysis.md)**

Static and dynamic analysis of HopHelper.exe. Extracted the C2 IP from PeStudio
string analysis, confirmed registry Run key persistence via Regshot comparison,
and verified active HTTP-based C2 communication through ProcMon TCP filtering.

---

**[Day 10 — SOC Alert Triage with Microsoft Sentinel](./03-Daily-Challenges/day-10-soc-alert-triage.md)**

Triaged 8 live incidents in Microsoft Sentinel — 4 high severity, 4 medium.
Wrote KQL queries against Linux syslog data to reconstruct a coordinated privilege
escalation campaign across four hosts: root SSH access, sudoers manipulation,
and kernel module persistence.

---

**[Day 15 — Web Attack Forensics](./03-Daily-Challenges/day-15-web-attack-forensics.md)**

Investigated a command injection attack by correlating Apache access logs, Apache
error logs, and Sysmon telemetry in Splunk. Confirmed OS-level code execution via
httpd.exe→cmd.exe parent-child relationship and decoded the Base64 payload.

---

**[Day 21 — HTA Malware Analysis](./03-Daily-Challenges/day-21-hta-malware.md)**

Static analysis of a malicious HTA file disguised as a salary survey. Traced the
VBScript execution chain from auto-load to fileless PowerShell execution, identified
the typosquatted C2 domain, and decoded the two-layer (Base64→ROT13) second-stage
payload in CyberChef.

---

**[Day 22 — C2 Detection with RITA and Zeek](./03-Daily-Challenges/day-22-c2-detection-rita.md)**

Proactive threat hunt — no active alerts. Converted PCAP to Zeek logs, imported
into RITA, and identified C2 beaconing to `rabbithole.malhare.net` through beacon
score and prevalence analysis alone. No signatures used.

---

## Skills Developed

**Blue Team (primary focus)**

SIEM operations across Splunk, Microsoft Sentinel, RITA, and Zeek. SOC alert triage
using a 4-factor model against real incident data. Malware analysis — static with
PeStudio and YARA, dynamic with ProcMon and Regshot, and static HTA analysis in a
text editor. Windows Registry forensics using Registry Explorer on offline hives.
C2 detection through behavioral analysis of network metadata. Phishing detection
via email header analysis, SPF/DKIM/DMARC verification, and punycode identification.
Introductory cloud security across Azure Sentinel and AWS IAM.

**Red Team (supporting knowledge)**

Web exploitation: XSS, IDOR, race conditions, cURL HTTP manipulation. Network
reconnaissance: Nmap, Netcat, service enumeration. Password cracking with John
the Ripper and pdfcrack. These days exist to understand what defenders need to catch,
not as a focus area.

**Emerging**

Prompt injection and agentic AI vulnerabilities (Days 4, 8). ICS/OT security via
Modbus TCP and pymodbus against a live PLC simulator (Day 19).

---

## Security+ SY0-701 Alignment

| Domain | Weight | Coverage |
|---|---|---|
| 1.0 General Security Concepts | 12% | Days 2, 5, 9, 11, 17, 18 |
| 2.0 Threats & Vulnerabilities | 22% | Days 2, 5, 6, 11–15, 18–24 |
| 3.0 Security Architecture | 18% | Days 7, 10, 14, 16, 19, 23 |
| 4.0 Security Operations | 28% | Days 3, 10, 13, 15, 16, 21, 22 |

Domains 2.0 and 4.0 together are 50% of the exam. Both are well-covered here.
Domain 5.0 (Security Program Management) has no meaningful coverage — this was
a technical hands-on series.

---

## Honest Assessment

Some days were well outside my comfort zone — the web security and
programming-heavy challenges especially. KQL and SPL required more reference
documentation than I expected. A few of the ICS and container days took multiple
attempts to follow properly.

What I found was that persistence and documentation mattered more than raw aptitude.
Writing up what I didn't fully understand the first time forced me to actually
understand it. The structured write-ups exist because of that, not despite it.

The consistent gap across SIEM work: query construction from scratch without
templates. I could follow investigation workflows and interpret results, but
independent query building is still developing. That's the honest version.

---

## Navigation

**For recruiters:**
Start here → `/02-Skills-Matrix/mapping.md` for the full technical inventory
with Security+ domain mapping and MITRE ATT&CK coverage →
`/04-Case-Studies/case-study.md` for the Blue Team investigation showcase →
daily challenge files for full write-ups with screenshots.

**Best days by topic:**
- SIEM / SOC: Days 3, 10, 15, 22
- Malware analysis: Days 6, 13, 21
- Forensics: Days 15, 16
- Network / C2: Days 7, 22

---

## Contact

**GitHub:** [uriel0byte](https://github.com/uriel0byte)  
**LinkedIn:** [Supawat H.](https://www.linkedin.com/in/supawat-h-145371392)  
**YouTube:** [@urielbyte](https://www.youtube.com/@urielbyte)  
**TryHackMe:** [poseidon.smash](https://tryhackme.com/p/poseidon.smash)

---

*Last updated: February 2026*
*Always learning, always cataloging the next threat.*
