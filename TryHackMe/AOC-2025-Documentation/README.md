# Advent of Cyber 2025
*uriel0byte — TryHackMe*

All 24 days completed. December 1–24, 2025.

This documents my full run through TryHackMe's Advent of Cyber 2025. As an ILS student
transitioning into cybersecurity, I used this as a structured introduction to SOC
operations. The write-ups are built for future reference, not just completion.

![AOC 2025 Certificate](./09-Certificates/aoc-2025-certificate.png)

**Stats at a glance**
- 24/24 days completed
- Days 1–10: enhanced with Key Takeaways sections
- 50+ tools used across 15+ security domains
- 60+ hours invested

---

## Repository Structure

```text
/AOC-2025-Documentation
├── README.md
├── /02-Skills-Matrix
│   └── mapping.md                   # Full skills inventory with Security+ domain mapping
├── /03-Daily-Challenges             # Core portfolio content
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
│   ├── day-12-phishing-detection.md
│   ├── day-13-yara-rules.md
│   ├── day-14-containers.md
│   ├── day-15-web-forensics.md
│   ├── day-16-registry-forensics.md
│   ├── day-17-cyberchef.md
│   ├── day-18-obfuscation.md
│   ├── day-19-ics-scada.md
│   ├── day-20-race-conditions.md
│   ├── day-21-hta-malware.md
│   ├── day-22-c2-detection.md
│   ├── day-23-aws-iam.md
│   └── day-24-curl-exploitation.md
├── /04-Case-Studies
│   └── case-study.md                # Index linking to case studies inside daily challenges
├── /07-Screenshots
└── /09-Certificates
    └── aoc-2025-certificate.pdf
```

---

## Challenge Overview

| Day | Title | Category | Difficulty | Key Tools |
|-----|-------|----------|------------|-----------|
| 1 | Shells Bells | Linux CLI | ★☆☆☆ | Bash, grep, find |
| 2 | Merry Clickmas | Phishing | ★★☆☆ | SET, SMTP |
| 3 | Did you SIEM? | Splunk SIEM | ★★★☆ | Splunk, SPL |
| 4 | old sAInt nick | AI Security | ★☆☆☆ | AI agents |
| 5 | Santa's Little IDOR | IDOR | ★★★☆ | DevTools, Burp |
| 6 | Egg-xecutable | Malware Analysis | ★★★☆ | PeStudio, ProcMon |
| 7 | Scan-ta Clause | Network Discovery | ★★☆☆ | Nmap, Netcat |
| 8 | Sched-yule conflict | Prompt Injection | ★★☆☆ | Agentic AI |
| 9 | A Cracking Christmas | Password Cracking | ★☆☆☆ | John, pdfcrack |
| 10 | Tinsel Triage | SOC Alert Triage | ★★★★ | MS Sentinel, KQL |
| 11 | Merry XSSMas | XSS | ★★☆☆ | Browser DevTools |
| 12 | Phishmas Greetings | Phishing Detection | ★★★☆ | Email headers |
| 13 | YARA mean one! | YARA Rules | ★★★☆ | YARA engine |
| 14 | DoorDasher's Demise | Containers | ★★★☆ | Docker |
| 15 | Drone Alone | Web Forensics | ★★★★ | Splunk, Sysmon |
| 16 | Registry Furensics | Windows Forensics | ★★★☆ | Registry Explorer |
| 17 | Hoperation Save McSkidy | CyberChef | ★★★☆ | CyberChef |
| 18 | The Egg Shell File | Obfuscation | ★★★☆ | CyberChef, PowerShell |
| 19 | Claus for Concern | ICS/SCADA | ★★★★ | Python, Modbus |
| 20 | Toy to The World | Race Conditions | ★★★☆ | Burp Suite |
| 21 | Malhare.exe | HTA Malware | ★★★★ | pluma, CyberChef |
| 22 | Command & Carol | C2 Detection | ★★★☆ | RITA, Zeek |
| 23 | S3cret Santa | AWS Security | ★★★☆ | AWS CLI |
| 24 | Hoperation Eggsploit | cURL Exploitation | ★★★☆ | cURL, bash |

★☆☆☆ Easy &nbsp;|&nbsp; ★★☆☆ Easy-Medium &nbsp;|&nbsp; ★★★☆ Medium-Hard &nbsp;|&nbsp; ★★★★ Hard

---

## Featured Write-ups

**[Day 3 — Splunk SIEM Log Analysis](./03-Daily-Challenges/day-03-splunk-siem.md)**

Investigated a ransomware attack using Splunk. Traced the full attack chain from
reconnaissance through SQL injection to data exfiltration, identified C2 communication,
and calculated 126KB of stolen data.

Skills: SPL query writing, log correlation, anomaly detection, attack chain reconstruction.

---

**[Day 6 — Malware Analysis](./03-Daily-Challenges/day-06-malware-analysis.md)**

Static and dynamic analysis of HopHelper.exe using PeStudio and ProcMon. Identified a
registry persistence mechanism, extracted the C2 IP from strings, and confirmed TCP-based
command-and-control communication.

Skills: Static analysis, dynamic sandbox methodology, IOC extraction, persistence detection.

---

**[Day 10 — SOC Alert Triaging with Microsoft Sentinel](./03-Daily-Challenges/day-10-soc-alert-triage.md)**

Simulated a full SOC analyst workflow. Triaged 8 incidents including 4 high-severity
privilege escalation alerts across 4 Linux hosts, used KQL to correlate events, and
identified kernel module persistence.

Skills: 4-factor triage model, KQL, cloud SIEM operations, multi-host incident correlation.

---

**[Day 15 — Web Attack Forensics](./03-Daily-Challenges/day-15-web-forensics.md)**

Investigated a command injection attack by correlating Apache access logs with Sysmon
events. Decoded a Base64 payload mid-chain and reconstructed the full attack timeline
from initial access to execution.

Skills: Multi-source log correlation, command injection analysis, Base64 decoding, attack
timeline reconstruction.

---

**[Day 22 — C2 Detection](./03-Daily-Challenges/day-22-c2-detection.md)**

Threat hunted for command-and-control beaconing using RITA and Zeek. Analyzed network
flows, identified suspicious DNS patterns and HTTP beaconing, and isolated C2
infrastructure.

Skills: RITA, Zeek, PCAP analysis, beacon detection, DNS tunneling identification.

---

## Skills Developed

### Blue Team (Primary Focus)
- SIEM operations: Splunk (SPL), Microsoft Sentinel (KQL), RITA, Zeek
- SOC alert triage: 4-factor model, multi-host incident correlation
- Malware analysis: static (PeStudio), dynamic (ProcMon, Regshot), YARA rules
- Forensics: Windows Registry (Registry Explorer), web attack forensics (Apache + Sysmon)
- C2 detection: beacon analysis, DNS tunneling, PCAP review
- Email security: phishing detection, SPF/DKIM/DMARC header analysis
- Cloud security: Azure Sentinel, AWS IAM enumeration, Docker container escape

### Red Team (Supporting Knowledge)
- Web exploitation: XSS, IDOR, race conditions
- Network reconnaissance: Nmap, Netcat, service enumeration
- Password cracking: John the Ripper, dictionary attacks
- Command-line exploitation: cURL, HTTP manipulation

### Emerging Areas
- AI security: prompt injection, agentic AI vulnerabilities (Days 4, 8)
- ICS/OT security: Modbus TCP, SCADA systems, pymodbus (Day 19)

---

## Security+ SY0-701 Alignment

| Domain | Weight | Coverage |
|--------|--------|----------|
| 1.0 General Security Concepts | 12% | Strong, Days 2, 5, 9, 11, 17, 18 |
| 2.0 Threats & Vulnerabilities | 22% | Excellent, Days 2, 5, 6, 11–15, 18–24 |
| 3.0 Security Architecture | 18% | Strong, Days 7, 10, 14, 16, 19, 23 |
| 4.0 Security Operations | 28% | Exceptional, Days 3, 10, 13, 15, 16, 21, 22 |

Domains 2.0 and 4.0 account for 50% of the exam. Both are well-covered here.

---

## Honest Assessment

Some days were well outside my comfort zone, the web security and
programming-heavy challenges especially. KQL and SPL required more reference
documentation than I expected. A few of the ICS and container days took multiple
attempts to follow properly.

What I found was that persistence and documentation mattered more than raw aptitude.
Writing up what I didn't fully understand the first time forced me to actually understand
it. The enhanced write-ups on Days 1–10 exist because of that, not despite it.

---

## Navigation

**For recruiters**
1. Start here for the overview
2. `/02-Skills-Matrix/mapping.md` for the full technical inventory
3. Days 3, 6, 10, 15, and 22 for the most relevant SOC demonstrations
4. `/07-Screenshots` for visual proof of work

**Best days by topic**
- SIEM / SOC: Days 3, 10, 15, 22
- Malware analysis: Days 6, 13, 21
- Forensics: Days 15, 16
- Network security: Days 7, 22, 24

---

## Contact

**GitHub:** [uriel0byte](https://github.com/uriel0byte)  
**LinkedIn:** [Supawat H.](https://www.linkedin.com/in/supawat-h-145371392)  
**YouTube:** [@urielbyte](https://www.youtube.com/@urielbyte)  
**TryHackMe:** [poseidon.smash](https://tryhackme.com/p/poseidon.smash)  

---

*Last updated: February 26, 2026*  
*Always learning, always cataloging the next threat.*
