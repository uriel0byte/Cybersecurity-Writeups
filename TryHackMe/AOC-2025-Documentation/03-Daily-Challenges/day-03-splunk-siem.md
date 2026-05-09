# Day 3: Did you SIEM? - Splunk Log Analysis

**Date:** December 3, 2025  
**Time Spent:** 3 hours  
**Difficulty:** ★★★☆  
**Category:** SIEM / Log Analysis / Defensive Security  
**Room:** https://tryhackme.com/room/splunkforloganalysis-aoc2025-x8fj2k4rqp

---

## Overview

King Malhare's Bandit Bunnies hit TBFC with a ransomware attack. A ransom message
appeared on the SOC dashboard demanding TBFC turn Christmas into "EAST-mas." Using
Splunk, I analyzed web traffic and firewall logs to trace the complete attack chain
from initial reconnaissance through SQL injection, webshell deployment, RCE, C2
communication, and data exfiltration.

First time using Splunk and SPL from scratch.

---

## What I Learned

### Splunk Interface

**Key components:**
- **Search and Reporting App** - Primary analyst interface
- **Search bar** - Where SPL queries are written, time range selector sits here
- **Timeline histogram** - Shows event distribution over time, reveals traffic spikes
- **Fields panel (left)** - Selected fields and interesting fields auto-extracted from logs
- **Events / Statistics / Visualization tabs** - Switch between raw events, aggregated
  stats, and charts

**Two data sources in this investigation:**

| Sourcetype | Contents | Key Fields |
|---|---|---|
| `web_traffic` | HTTP requests to/from web server | `client_ip`, `user_agent`, `path`, `status` |
| `firewall_logs` | Firewall allow/block events | `src_ip`, `dest_ip`, `action`, `dest_port`, `bytes_transferred`, `reason` |

Web server local IP: **10.10.1.5**  
Attacker IP: **198.51.100.55**

---

## SPL Reference (Core Syntax)

```spl
# Basic query
index=main sourcetype=web_traffic

# Boolean operators
AND   # Both conditions must match
OR    # Either condition matches
NOT   # Exclude matches

# Comparison operators
=     # Exact match
!=    # Exclude
>     # Greater than
<     # Less than

# Wildcards
*     # Any characters
?     # Single character

# Examples
user_agent=*sqlmap*        # Contains "sqlmap"
user_agent!=*Mozilla*      # Does NOT contain "Mozilla"
status>=400                # Status 400 or higher
```

**IN operator:**
```spl
path IN ("/.env", "/.git*", "/*phpinfo*")
# Matches any of the listed values
```

**Pipe chaining:**
```spl
index=main | command1 | command2 | command3
# Results flow left to right
```

### Critical SPL Commands

**timechart - visualize events over time:**
```spl
index=main | timechart span=1d count
# span=1d  → group by day
# span=1h  → group by hour
# span=15m → group by 15 minutes
```

**stats - aggregate and calculate:**
```spl
| stats count by client_ip
| stats sum(bytes_transferred) by src_ip
| stats count, avg(response_time), max(bytes) by host
# Other functions: min(), avg(), median(), values(), dc()
```

**table - display specific fields:**
```spl
| table _time, client_ip, path, status
```

**sort - order results:**
```spl
| sort -count        # Descending (- prefix)
| sort count         # Ascending
```

**head / tail:**
```spl
| head 10    # First 10 results
| tail 20    # Last 20 results
```

### Field Names by Source

**Critical:** Field names differ between sources. Using the wrong one returns nothing.

| web_traffic | firewall_logs |
|---|---|
| `client_ip` | `src_ip` |
| `user_agent` | `action` (ALLOWED/BLOCKED) |
| `path` | `dest_ip` |
| `status` | `bytes_transferred` |
| | `reason` |

---

## Attack Chain Investigation

### Phase 1 - Reconnaissance

```spl
sourcetype=web_traffic client_ip="198.51.100.55"
AND path IN ("/.env", "/*phpinfo*", "/.git*")
| table _time, path, user_agent, status
```

**Findings:**
- Tools used: `curl`, `wget`
- Targets: `/.env`, `/phpinfo.php`, `/.git/config`
- HTTP responses: 404, 403, 401 (files secured or nonexistent)
- Probing for common misconfigurations before moving to active exploitation

### Phase 2 - Enumeration

```spl
sourcetype=web_traffic client_ip="198.51.100.55"
AND path="*..\/..\/*" OR path="*redirect*"
| stats count by path
```

**Findings:**
- Path traversal attempts: `../../etc/passwd`, `../../windows/system32/`
- Open redirect testing via URLs with `redirect` parameter
- Active vulnerability testing, no longer passive scanning

### Phase 3 - Exploitation (SQL Injection)

```spl
sourcetype=web_traffic client_ip="198.51.100.55"
AND user_agent IN ("*sqlmap*", "*Havij*")
| table _time, path, status
```

**Findings:**
- `sqlmap` and `Havij` user agents confirmed
- `SLEEP(5)` payloads visible in path
- HTTP **504 Gateway Timeout** responses

**Why 504 confirms SQL injection:**  
`SLEEP(5)` tells the database to pause 5 seconds. If the server times out, the
query executed. This is time-based blind SQL injection — the attacker infers
success from response time rather than visible output.

### Phase 4 - Data Exfiltration

```spl
sourcetype=web_traffic client_ip="198.51.100.55"
AND path IN ("*backup.zip*", "*logs.tar.gz*")
| table _time, path, user_agent
```

**Findings:**
- `backup.zip` and `logs.tar.gz` downloaded via `curl` and `zgrab`
- Compressed archives likely contain credentials, session tokens, database content
- Sets up double-extortion: data theft before deploying ransomware

### Phase 5 - Ransomware Staging and RCE

```spl
sourcetype=web_traffic client_ip="198.51.100.55"
AND path IN ("*bunnylock.bin*", "*shell.php?cmd=*")
| table _time, path, user_agent, status
```

**Findings:**
- `shell.php` webshell uploaded and active
- `shell.php?cmd=./bunnylock.bin` - ransomware executed through webshell
- `bunnylock.bin` is the ransomware payload

**What a webshell is:**  
A malicious script (PHP, JSP, ASP) uploaded to the web server. The `?cmd=`
parameter lets the attacker run any system command remotely. Full RCE.

**Attack progression:**  
SQL injection gained database access → webshell uploaded via SQLi → commands
executed through webshell → ransomware binary ran.

### Phase 6 - C2 Communication (Firewall Pivot)

```spl
sourcetype=firewall_logs src_ip="10.10.1.5"
AND dest_ip="198.51.100.55" AND action="ALLOWED"
| table _time, action, protocol, src_ip, dest_ip, dest_port, reason
```

**Pivot note:** Switching from `web_traffic` to `firewall_logs` here. The compromised
server IP (10.10.1.5) becomes the source, and the attacker IP becomes the destination.

**Findings:**
- `ACTION=ALLOWED` - Firewall permitted the connection
- `REASON=C2_CONTACT` - Explicitly labeled command and control
- Outbound connection from server to attacker infrastructure
- Ransomware reporting back: sends encryption keys, system info, awaits commands

### Phase 7 - Quantify Exfiltration

```spl
sourcetype=firewall_logs src_ip="10.10.1.5"
AND dest_ip="198.51.100.55" AND action="ALLOWED"
| stats sum(bytes_transferred) by src_ip
```

**Result:** **126,167 bytes (~126KB)** transferred from compromised server to attacker.

---

## Complete Attack Chain Summary

| Phase | Indicator | SPL Focus |
|---|---|---|
| Reconnaissance | `/.env`, `/.git`, `/phpinfo.php` via curl/wget | `path IN`, `user_agent` |
| Enumeration | Path traversal `../../`, open redirect | `path="*..\/..\/*"` |
| Exploitation | sqlmap, SLEEP(5), HTTP 504 | `user_agent IN ("*sqlmap*")` |
| Exfiltration | backup.zip, logs.tar.gz downloads | `path IN ("*backup.zip*")` |
| RCE | shell.php webshell, bunnylock.bin | `path="*shell.php?cmd=*"` |
| C2 | Outbound ALLOWED, REASON=C2_CONTACT | `sourcetype=firewall_logs` pivot |
| Impact | 126,167 bytes (~126KB) transferred | `stats sum(bytes_transferred)` |

---

## Challenges

First exposure to Splunk and SPL. The hardest part was not the interface but
understanding when to pivot between data sources and why field names change between
`web_traffic` and `firewall_logs`. Trying to use `client_ip` in a firewall query
returns nothing because that source uses `src_ip`.

The pipe operator took adjustment — understanding that results flow left to right and
each command transforms the output of the previous one. Once that clicked, building
complex queries became more logical.

17,172 events looked unmanageable at first. The timeline visualization made it
tractable immediately by narrowing the attack window visually before writing a
single filter.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** SQL injection,
webshells, ransomware, path traversal, reconnaissance methods, C2 communication.

**Domain 3.0 - Security Architecture (18%):** Log aggregation, security monitoring
architecture, data source correlation.

**Domain 4.0 - Security Operations (28%):** SIEM usage, log analysis, alert triage,
threat hunting, incident investigation, attack timeline reconstruction.

---

## Evidence

![traffic-spike-timeline](../07-Screenshots/Day3-1.png)
*Traffic spike identified using timechart. The volume increase on the attack date
narrowed the investigation window before any filtering was applied.*

![attacker-ip-correlation](../07-Screenshots/Day3-2.png)
*Attacker IP 198.51.100.55 isolated by filtering out legitimate browser user agents
and running stats count by client_ip. Responsible for 7,876 suspicious requests
including path traversal, SQL injection probes, and webshell uploads.*

![c2-firewall-logs](../07-Screenshots/Day3-3.png)
*Firewall logs confirming outbound C2 connection from compromised server (10.10.1.5)
to attacker infrastructure (198.51.100.55). ACTION=ALLOWED, REASON=C2_CONTACT.*

![sum-c2-firewall-logs](../07-Screenshots/Day3-4.png)
*Total bytes transferred calculated using stats sum(bytes_transferred). Result:
126,167 bytes (~126KB) exfiltrated from the compromised server.*

---

## Key Takeaways

**Investigation workflow template:**

```spl
# Step 1: Orient
index=main | timechart span=1h count

# Step 2: Filter benign
sourcetype=web_traffic
user_agent!=*Mozilla* user_agent!=*Chrome*
user_agent!=*Safari* user_agent!=*Firefox*

# Step 3: Find top attacker IPs
| stats count by client_ip | sort -count | head 5

# Step 4: Focus on attacker
sourcetype=web_traffic client_ip="198.51.100.55"
| table _time, path, user_agent, status

# Step 5: Trace phases
# Recon
AND path IN ("/.env", "/.git*")
# Exploitation
AND user_agent IN ("*sqlmap*", "*Havij*")
# Exfiltration
AND path IN ("*backup.zip*", "*logs.tar.gz*")
# RCE
AND path="*shell.php?cmd=*"

# Step 6: Pivot to firewall
sourcetype=firewall_logs src_ip="10.10.1.5"
AND dest_ip="198.51.100.55" AND action="ALLOWED"
| table _time, action, protocol, dest_port, reason

# Step 7: Quantify
| stats sum(bytes_transferred) by src_ip
```

**Suspicious user agents to hunt:**

| Category | Agents |
|---|---|
| SQL injection | `sqlmap`, `Havij`, `jSQL` |
| Web scanners | `nikto`, `dirb`, `gobuster`, `wfuzz` |
| Command-line | `curl`, `wget`, `python-requests` |
| Network scanners | `zgrab`, `masscan`, `nmap` |

Context matters — `curl` is not always malicious, but non-browser agents warrant
investigation.

**HTTP status codes worth knowing:**

| Code | Meaning | Attack context |
|---|---|---|
| 200 | OK | Request succeeded |
| 401 | Unauthorized | Auth required, probing |
| 403 | Forbidden | No permission |
| 404 | Not Found | Recon attempt |
| 504 | Gateway Timeout | Often confirms blind SQL injection |

**Common SPL mistakes:**

```spl
# Missing wildcard
user_agent=sqlmap        # Only matches exact string
user_agent=*sqlmap*      # Correct

# Wrong field name between sources
client_ip  # web_traffic
src_ip     # firewall_logs (not the same)

# Missing pipe
index=main stats count   # Wrong
index=main | stats count # Correct

# Case sensitivity
client_IP  # Wrong
client_ip  # Correct
```

**Log correlation pivot logic:**
- Suspicious IP in web logs → pivot to firewall logs with that IP as `dest_ip`
- Compromised server in firewall logs → pivot to web logs with that IP as `client_ip`
- Malware indicator in endpoint logs → pivot to network logs for C2 traffic
