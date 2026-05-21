# Case Studies — Blue Team Investigations

This folder indexes the Blue Team investigation work from Advent of Cyber 2025.
Each entry below is a summary of what was investigated, what tools were used,
and what the outcome was. Full writeups with screenshots, queries, and room
answers live in `/03-Daily-Challenges/`.

These are the days most directly relevant to SOC Tier 1 work: SIEM triage,
malware analysis, log correlation, phishing investigation, forensics, and
threat hunting.

---

## Core SOC Investigations

### Day 10 — SOC Alert Triage (Microsoft Sentinel)
`/03-Daily-Challenges/day-10-soc-alert-triage.md`

**What happened:** 8 alerts fired across TBFC's Azure environment during an
active intrusion. 4 high, 4 medium severity. Task was to triage in priority
order, investigate each, and determine the full attack chain.

**What I did:** Used Microsoft Sentinel's Incidents tab to sort by severity,
investigated each alert's evidence section, and wrote KQL queries against
`Syslog_CL` to reconstruct attacker activity across four Linux hosts.

**Finding:** Coordinated campaign — all four hosts showed the same TTP
progression. Root SSH access → SUID discovery → sudoers manipulation →
`malicious_mod.ko` kernel module insertion for boot persistence. Single
attacker, four simultaneous footholds.

**Why it matters for SOC Tier 1:** This is the literal job. Multi-alert
triage, correlation across hosts, escalation decision-making.

---

### Day 3 — SIEM Investigation (Splunk)
`/03-Daily-Challenges/day-03-splunk-siem.md`

**What happened:** Ransomware hit TBFC's web server. Task was to trace the
complete attack from first probe to ransomware deployment using Splunk.

**What I did:** Wrote SPL queries across `web_traffic` and `firewall_logs`
sourcetypes to trace seven distinct attack phases. Pivoted from web logs
to firewall logs to follow the attacker from initial access to C2 callback.

**Finding:** Full chain — reconnaissance with curl/wget → path traversal
enumeration → SQL injection (confirmed by 504 timeout on `SLEEP(5)`) →
data exfiltration of backup.zip → webshell upload → ransomware execution →
outbound C2 (`REASON=C2_CONTACT`) → 126KB exfiltrated.

**Why it matters for SOC Tier 1:** Attack chain reconstruction from raw SIEM
data. Pivoting between log sources. The 7-phase structure maps directly to
real incident investigation methodology.

---

### Day 15 — Web Attack Forensics (Splunk + Sysmon)
`/03-Daily-Challenges/day-15-web-attack-forensics.md`

**What happened:** TBFC's drone scheduler was receiving long HTTP requests
with Base64 payloads. Sysmon alerted: Apache spawned an unusual process.

**What I did:** Investigated across three Splunk indexes — Apache access
logs, Apache error logs, and Sysmon — using a 5-step methodology to prove
each stage of the attack.

**Finding:** Command injection via `/cgi-bin/hello.bat?cmd=powershell`.
HTTP 500 confirmed backend execution. Sysmon confirmed `httpd.exe` spawned
`cmd.exe` — OS-level code execution. Attacker ran `whoami` immediately after.
Encoded PowerShell payload (`MUAHAHAA`) visible in access logs but never
executed on host — defenses held at that stage.

**Why it matters for SOC Tier 1:** Multi-source correlation is the hardest
part of web attack investigation. Access logs alone don't prove exploitation —
Sysmon's parent-child process data does.

---

### Day 22 — C2 Detection & Threat Hunting (RITA + Zeek)
`/03-Daily-Challenges/day-22-c2-detection-rita.md`

**What happened:** No active alerts — proactive hunt during a quiet period.
Task was to find any C2 communication that had slipped through without
triggering signatures.

**What I did:** Converted two PCAPs to structured Zeek logs, imported into
RITA, and hunted using beacon score, prevalence, and connection metadata
filters. No signatures — pure behavioral detection.

**Finding:** `rabbithole.malhare.net` flagged with high beacon score, low
prevalence (few internal hosts), and rare TLS signature. C2 beaconing
confirmed through timing analysis of encrypted traffic without decrypting
any payload.

**Why it matters for SOC Tier 1:** Modern C2 is encrypted. Signature-based
tools miss it. RITA's behavioral approach catches what rules don't — and
this is where SOC analysts provide value that automated tools can't.

---

## Malware & Forensic Investigations

### Day 21 — HTA Malware Analysis (Static)
`/03-Daily-Challenges/day-21-hta-malware.md`

**What happened:** TBFC employees received a phishing email with a fake
salary survey — an HTA file that executed silently while the user filled in
a form.

**What I did:** Opened `survey.hta` in a text editor only. Mapped the
VBScript execution chain: `window_onLoad` → `getQuestions()` (C2 download)
→ `provideFeedback()` (PowerShell execution). Decoded two-layer payload
(Base64 → ROT13) in CyberChef to reveal the fileless execution chain.

**Finding:** Auto-execution on open. Exfiltrated `ComputerName` and
`UserName` via GET to `survey.bestfestiivalcompany.com/details` (typosquatted
domain). Silent PowerShell execution: `-nop -w hidden`. Second-stage payload
decoded to `$U → $C → $B → Invoke-Command` — fileless, entirely in memory.

**Why it matters for SOC Tier 1:** Living off the Land attacks using `mshta.exe`
are active in real campaigns. Being able to read VBScript and decode payloads
without running the file is a practical IR skill.

---

### Day 6 — Malware Analysis (Static + Dynamic)
`/03-Daily-Challenges/day-06-malware-analysis.md`

**What happened:** Suspicious executable arrived via phishing email at 3AM.
Claimed to be a scheduling program.

**What I did:** Phase 1 static analysis with PeStudio — extracted SHA256,
found embedded C2 IP `138.62.51.186` in strings, flagged suspicious imports
(`RegSetValueEx`, `WSAConnect`). Phase 2 dynamic analysis in sandbox —
Regshot confirmed Run key persistence, ProcMon confirmed active HTTP C2
connections.

**Finding:** `HopHelper.exe` persists via
`HKCU\Software\Microsoft\Windows\CurrentVersion\Run` and communicates over
HTTP to `138.62.51.186`. Attacker has persistent remote access after each
reboot.

**Why it matters for SOC Tier 1:** Static analysis finds what the file can
do. Dynamic analysis confirms what it actually does. IOCs extracted go
straight into firewall blocks, EDR rules, and SIEM alerts.

---

### Day 16 — Windows Registry Forensics
`/03-Daily-Challenges/day-16-registry-forensics.md`

**What happened:** `dispatch-srv01` was compromised starting October 21,
2025. No obvious malware file. Registry hives pulled from the system for
offline analysis.

**What I did:** Loaded SOFTWARE and NTUSER.DAT hives into Registry Explorer
(SHIFT+Open to replay transaction logs). Checked Uninstall key for suspicious
installs, UserAssist for execution evidence, Run key for persistence.

**Finding:** `DroneManager Updater` installed pre-compromise date. Launched
from `C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe` (direct
internet download, not IT deployment). Persistent via Run key as
`dronehelper.exe --background` — silent execution on every login.

**Why it matters for SOC Tier 1:** Registry is often the only forensic
artifact that survives after a malware cleanup. Knowing which hive holds
which evidence (SOFTWARE for installs/persistence, NTUSER.DAT for user
execution history) is fundamental to Windows IR.

---

## Supporting Blue Team Skills

These days cover techniques that directly support SOC work even though
they are not pure investigation rooms.

**Day 12 — Phishing Analysis:** Email header analysis, SPF/DKIM/DMARC
verification, punycode detection, legitimate-platform phishing flows.
Daily triage skill for any SOC analyst.

**Day 13 — YARA Rules:** Wrote custom rules from scratch using regex and
hex patterns. Post-incident sweep methodology — malware found on one host,
scan everything else for the same IOC.

**Day 9 — Password Cracking:** Detection methodology for offline cracking
on Windows and Linux endpoints. Sysmon event patterns and auditd rules that
catch `john`/`hashcat` execution before cracking completes.

**Day 18 — Obfuscation:** Two-direction exercise — deobfuscated a C2 URL
from `SantaStealer.ps1`, then obfuscated an API key with XOR. Understanding
how PowerShell malware hides payloads is prerequisite knowledge for any
malware triage.

**Day 4 — AI in Security:** AI-assisted log analysis workflow. Practical
context for how SIEM vendors are embedding AI into alert triage — useful
for understanding where the tooling is heading.

---

## What Is Not Covered Here

Days 2, 5, 7, 8, 11, 14, 17, 19, 20, 23, 24 covered offensive techniques
(phishing campaigns, IDOR, XSS, container escape, race conditions, AWS
enumeration, cURL exploitation) and specialized areas (ICS/Modbus,
prompt injection). Those writeups are in `/03-Daily-Challenges/` and are
documented for completeness, but they are not the focus of this Blue Team
profile.

Understanding offensive techniques makes defensive detection rules better.
But they are not the primary skillset being built here.

---
