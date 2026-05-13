# Day 10: SOC Alert Triage — Tinsel Triage

**Date:** December 10, 2025  
**Time Spent:** 3 hours  
**Difficulty:** ★★★★ *(Official rating: Medium — rated higher; Sentinel, Azure, and KQL were all new simultaneously)*  
**Category:** SIEM / SOC Operations / Cloud Security  
**Room:** https://tryhackme.com/room/azuresentinel-aoc2025-a7d3h9k0p2

---

## Overview

The Best Festival Company's SOC was flooded with alerts — Evil Easter Bunnies were
hitting the Azure environment hard. Playing McSkidy, I triaged 8 incidents (4 high,
4 medium severity) in Microsoft Sentinel, investigated privilege escalation attacks
across four Linux hosts (app-01, app-02, websrv-01, storage-01), and used KQL to
reconstruct complete attack chains from initial root SSH access through SUID discovery,
sudoers manipulation, and kernel module persistence.

First exposure to Microsoft Sentinel and KQL.

---

## What I Learned

### Why Alert Triage Matters

When alerts flood in, working them sequentially is how you miss the one that matters.
Some are noise, some are false positives needing rule tuning, and a small number are
active threats that need immediate action. Triage is the process of sorting those
buckets before touching a single alert in depth.

### The 4-Factor Triage Model

| Factor | Question | Action |
|---|---|---|
| Severity | How bad? | Start with Critical/High |
| Timestamp & Frequency | When? Repeating? | Build timeline, spot ongoing attacks |
| Attack Stage | Where in the kill chain? | Understand attacker progression |
| Affected Asset | What got hit? | Prioritize by business criticality |

### Microsoft Sentinel Interface

**Key components:**
- **Incidents tab** (Threat Management) — centralized dashboard, severity filtering,
  timeline of incident creation
- **Alert Details** — event count, entities involved, MITRE ATT&CK tactic, creation
  timestamp, "View full details" button
- **Evidence section** — raw event data behind the alert, actual commands executed,
  logs from affected systems
- **Log Analytics** — custom KQL queries against log tables (e.g., `Syslog_CL`)
- **Similar Incidents** — Sentinel links related alerts by shared entity for correlation

**Navigation path:**
```
Azure Portal → Microsoft Sentinel → Threat Management → Incidents tab
→ Click incident → right panel → "View full details"
→ Evidence section → Events → switch to KQL mode
```

### KQL (Kusto Query Language)

KQL is Sentinel's query language. Syntax is different from SPL but the pipe logic
is the same — results flow left to right, each command transforms the previous output.

**Basic query structure:**
```kql
set query_now = datetime(2025-10-30T05:09:25.9886229Z);
Syslog_CL
| where host_s == 'app-02'
| project _timestamp_t, host_s, Message
```

Breaking it down:
```
set query_now =    Sets time reference point for the query
Syslog_CL          Custom Linux syslog table (_CL suffix = custom log)
| where            Filter rows
| project          Select which columns to display
host_s             String field (_s suffix = string type in custom logs)
_timestamp_t       Datetime field (_t suffix = datetime type)
```

**Core operators:**

| Operator | Purpose | Example |
|---|---|---|
| `where` | Filter rows | `where host_s == 'app-02'` |
| `project` | Select columns | `project _timestamp_t, Message` |
| `sort by` | Order results | `sort by _timestamp_t desc` |
| `take` | Limit rows | `take 10` |
| `summarize count()` | Count records | `summarize count() by host_s` |
| `contains` | Substring match | `where Message contains 'sudo'` |
| `datetime()` | Time value | `datetime(2025-10-30T05:09:25Z)` |

**KQL vs SPL — key differences:**
```
KQL:  | where host_s == 'app-02'
SPL:  client_ip="198.51.100.55"

KQL:  | project _timestamp_t, Message
SPL:  | table _time, path

KQL:  | summarize count() by host_s
SPL:  | stats count by client_ip

KQL:  | sort by _timestamp_t desc
SPL:  | sort -count
```

### 6-Step Investigation Process

```
1. Investigate alert details
   → Entities, event data, detection logic
   → Confirm malicious vs. benign

2. Check related logs
   → Raw log sources behind the alert
   → Look for corroborating patterns

3. Correlate across alerts
   → Same host, user, or source IP in other alerts?
   → Broader attack sequence?

4. Build timeline
   → Combine timestamps and actions
   → Ongoing or already contained?

5. Decide action
   → Escalate (IOCs confirmed)
   → Investigate further (need more evidence)
   → Close/suppress (confirmed false positive, tune the rule)

6. Document
   → Analysis, decision, remediation steps
```

### Attack Chain Investigation

**Affected hosts:** app-01, app-02, websrv-01, storage-01

All four showed the same TTP pattern, indicating a coordinated campaign rather than
isolated incidents. Multiple alerts per host are not separate problems — they are
different stages of one intrusion.

**Full attack chain reconstructed:**
```
External SSH → Root access gained
      ↓
SUID discovery → Enumerating escalation paths
      ↓
User added to sudoers → Backup admin account created (Alice)
      ↓
Shadow file copied → Credential harvesting staged
      ↓
Kernel module inserted → malicious_mod.ko loaded for boot persistence
      ↓
Attacker survives reboots, maintains access
```

**app-02 KQL findings (chronological):**

| Event | What It Means |
|---|---|
| `cp /etc/shadow` executed | Shadow file backed up for credential harvest |
| User Alice added to sudoers | Admin rights granted to attacker-controlled account |
| backupuser modified by root | Second persistence account created |
| `malicious_mod.ko` inserted | Kernel module loaded — persists across reboots |
| Root SSH authentication confirmed | Initial access entry point |

**Alert-to-stage mapping (app-02 example):**

| Alert | MITRE Tactic | What It Reveals |
|---|---|---|
| Root SSH Login from External IP | Initial Access | Attacker gained remote access |
| SUID Discovery | Privilege Escalation | Searching for escalation paths |
| Kernel Module Insertion | Persistence | Survival across reboots confirmed |

**High-severity alerts triaged:**
- Linux PrivEsc — Kernel Module Insertion
- Linux PrivEsc — Polkit Exploit Attempt
- Linux PrivEsc — Sudo Shadow Access
- Linux PrivEsc — User Added to Sudo Group

---

## Challenges

Everything here was new. Sentinel's interface, Azure portal navigation, KQL syntax,
and cloud SIEM workflows all at once in three hours. KQL was the hardest part —
Day 3 SPL knowledge had mostly faded by Day 10, so starting a second query language
from scratch without a programming background made the first queries rough.

Alert correlation took the longest to click. Treating three alerts on the same host
as one attack rather than three separate problems is the right instinct, but it was
not obvious until the attack chain started taking shape from the evidence.

ProcMon from Day 6 captured too many events — Sentinel had the opposite problem:
correlating across sparse, high-level alerts required more inference. Both extremes
are real SOC challenges. Lab credentials expired before screenshots could be captured.

The room is officially rated Medium. That label makes sense in isolation, but hitting
Sentinel, Azure navigation, and KQL simultaneously with no prior exposure to any of
them pushed the actual difficulty higher.

---

## Security+ Alignment

**Domain 3.0 - Security Architecture (18%):** Cloud security architecture, SIEM
deployment, security monitoring infrastructure.

**Domain 4.0 - Security Operations (28%):** SIEM operations, alert triage, incident
investigation, log analysis, threat hunting, incident response, documentation.

---

## Key Takeaways

**4-factor triage model:**
```
Severity    → Start with Critical/High
Time        → Is this ongoing?
Stage       → How far has the attacker gotten?
Asset       → How critical is what got hit?
```

**KQL quick reference:**
```kql
// Basic filter and display
Syslog_CL
| where host_s == 'hostname'
| project _timestamp_t, host_s, Message

// Count by host
Syslog_CL
| summarize count() by host_s

// Substring search
Syslog_CL
| where Message contains 'sudo'
| sort by _timestamp_t desc
| take 20
```

**Linux privilege escalation commands to flag:**
```bash
find / -perm -4000              # SUID binary discovery
usermod -aG sudo <user>         # Add user to sudoers
cp /etc/shadow                  # Shadow file backup
insmod <module>.ko              # Kernel module insertion
modprobe <module>               # Kernel module loading
echo '<user> ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

**Files to monitor on Linux hosts:**
```
/etc/passwd           User accounts
/etc/shadow           Password hashes (access = credential harvest risk)
/etc/sudoers          Sudo permissions
/etc/ssh/sshd_config  SSH configuration
/lib/modules/         Kernel modules
```

**Entity correlation logic:**
```
Multiple alerts on same host/user/IP?
→ Check timestamps (close = same attack)
→ Map to kill chain stages
→ If progression visible (access → escalation → persistence): ESCALATE
```

**Splunk (SPL) vs. Sentinel (KQL) at a glance:**

| Action | SPL | KQL |
|---|---|---|
| Filter | `field="value"` | `\| where field == 'value'` |
| Select columns | `\| table field1, field2` | `\| project field1, field2` |
| Count by field | `\| stats count by field` | `\| summarize count() by field` |
| Order results | `\| sort -count` | `\| sort by field desc` |
| Limit rows | `\| head 10` | `\| take 10` |
