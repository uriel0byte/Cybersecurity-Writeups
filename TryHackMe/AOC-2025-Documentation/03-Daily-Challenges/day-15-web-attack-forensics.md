# Day 15: Web Attack Forensics — Drone Alone

**Date:** December 15, 2025  
**Time Spent:** 1.5 hours  
**Difficulty:** ★★★★ *(Official rating: Easy — rated harder; query crafting from scratch and multi-source correlation were both new)*  
**Category:** SIEM / Incident Response / Web Attack Forensics / Blue Team  
**Room:** https://tryhackme.com/room/webattackforensics-aoc2025-b4t7c1d5f8

---

## Overview

TBFC's drone scheduler web UI started receiving strange, long HTTP requests with
Base64-encoded chunks. Sysmon raised an alert: Apache spawned an unusual process.
Using Splunk, I triaged the incident by pivoting between Apache access logs, Apache
error logs, and Sysmon telemetry — detecting the command injection vector, confirming
OS-level code execution, tracking post-exploitation recon, and decoding obfuscated
PowerShell payloads to reconstruct the full attack chain.

Favorite room so far. This is exactly the work I want to do.

---

## What I Learned

### Investigation Workflow

The core methodology for web attack forensics. This order matters — each step
answers a specific question before moving to the next.

| Step | Log Source | Question |
|---|---|---|
| 1 | Apache Access Logs | Did the attack reach the server? |
| 2 | Apache Error Logs | Did the server try to execute it? |
| 3 | Sysmon Process Logs | Did the OS actually run something? |
| 4 | Sysmon Command History | What did the attacker do after? |
| 5 | Base64 Decode | What was the actual payload? |

### Step 1 — Detect Web Command Injection (Access Logs)

```spl
index=windows_apache_access (cmd.exe OR powershell OR "powershell.exe" OR "Invoke-Expression")
| table _time host clientip uri_path uri_query status
```

Searches Apache access logs for requests containing command execution indicators.
The payload hides in `uri_query` as a Base64-encoded string.

**Finding:** HTTP requests to `/cgi-bin/hello.bat?cmd=powershell` with long encoded
strings in the query parameter.

**Example encoded payload:**
```
VABoAGkAcwAgAGkAcwAgAG4AbwB3ACAATQBpAG4AZQAhACAATQBVAEEASABBAEEASABBAEEA
```

**Decoded:** `This is now Mine! MUAHAHAA`

**Tool:** https://www.base64decode.org/

**Note:** PowerShell's `-EncodedCommand` flag uses **UTF-16LE** encoding, not
standard UTF-8 Base64. Attackers encode payloads to evade signature-based detection
— the string looks like random noise and won't match known malicious patterns.

### Step 2 — Confirm Backend Execution (Error Logs)

```spl
index=windows_apache_error ("cmd.exe" OR "powershell" OR "Internal Server Error")
```

**View setting:** Switch to `Raw` in the event display dropdown above the results.

HTTP 500 on a CGI endpoint that received a PowerShell command means the server
processed the malicious input and attempted execution. This is the dividing line
between a blocked request (web layer) and exploitation reaching the backend.

### Step 3 — Verify OS-Level Code Execution (Sysmon)

```spl
index=windows_sysmon ParentImage="*httpd.exe"
```

**View setting:** Switch to `Table` in the event display dropdown.

Apache (`httpd.exe`) should only spawn worker threads. Finding `cmd.exe` as a
child process is unambiguous confirmation of successful exploitation.

**Malicious finding:**
```
ParentImage: C:\Apache24\bin\httpd.exe
Image:       C:\Windows\System32\cmd.exe
```

This is the hardest evidence in the chain. Incident severity is HIGH/CRITICAL
from this point forward.

### Step 4 — Track Post-Exploitation Reconnaissance

```spl
index=windows_sysmon *cmd.exe* *whoami*
```

`whoami` is the first command attackers run after gaining execution. It confirms
which user context they landed in and determines privilege level:

| Result | Meaning |
|---|---|
| `NT AUTHORITY\SYSTEM` | Full control |
| `IIS APPPOOL\DefaultAppPool` | Web service account — limited but pivotable |
| Standard user | Restricted foothold |

Finding `whoami` in Sysmon confirms the attack fully succeeded and the attacker
is actively enumerating — not just probing.

**Kill chain at this point:**
```
Command Injection → Code Execution → OS Recon → [Privilege Esc / Lateral Movement / Persistence]
```

### Step 5 — Hunt Encoded PowerShell Payloads

```spl
index=windows_sysmon Image="*powershell.exe" (CommandLine="*enc*" OR CommandLine="*-EncodedCommand*" OR CommandLine="*Base64*")
```

Searches Sysmon for PowerShell execution events where the command line contains
encoding flags.

**This room's result: no results.** The attacker's encoded payload was visible
in the HTTP request (Step 1), but PowerShell with an encoded command never
actually executed on the host OS. The attempt was made — execution was not
completed. That is the correct Blue Team outcome.

**If this query returns results in other investigations:** decode every Base64
string found and document. In real attacks, encoded payloads are reverse shells,
credential dumpers, persistence mechanisms, or ransomware droppers. The encoding
is cosmetic — it comes off in one step.

### Command Injection: How It Works

The vulnerable endpoint: `/cgi-bin/hello.bat?cmd=<input>`

The script passes the `cmd` parameter directly to the system shell with no
validation. Attacker substitutes a benign value with `powershell -enc <BASE64>`.
Server executes it.

**Root cause:** No input validation, no command whitelisting, CGI script trusted
user input unconditionally.

---

## Challenges

The investigation logic and reading query results clicked immediately. What didn't:
writing queries from scratch. This room provided every query — in a real SOC, they
won't exist. I could follow the methodology and interpret findings but couldn't
build these independently cold.

Specific gaps: Sysmon field names (`Image`, `ParentImage`, `CommandLine`) and
knowing which index holds which log type. That's a reps problem. More cold labs
are the fix.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Command injection,
web application attacks, obfuscation techniques, reconnaissance.

**Domain 4.0 - Security Operations (28%):** SIEM operations, log analysis, incident
response, forensic investigation, threat hunting, alert triage.

---

## Evidence

![Splunk Command Injection Detection](../07-Screenshots/Day15-1.png)
*Detected command injection attempts in Apache access logs — Base64-encoded payloads
visible in the uri_query field.*

![Sysmon Process Tree Analysis](../07-Screenshots/Day15-2.png)
*httpd.exe spawning cmd.exe confirmed successful command injection and OS-level
code execution.*

![Base64 Payload Decoding](../07-Screenshots/Day15-3.png)
*Decoded Base64 payload: "This is now Mine! MUAHAHAA" — attacker message confirming
full execution.*

---

## Key Takeaways

**Investigation workflow template:**
```
Web Access Logs → Error Logs → Sysmon Process Logs → Recon Commands → Payload Decoding
```

**Splunk query reference:**
```spl
-- Detect command injection in web requests
index=windows_apache_access (cmd.exe OR powershell OR "Invoke-Expression")
| table _time host clientip uri_path uri_query status

-- Confirm backend execution
index=windows_apache_error ("cmd.exe" OR "powershell" OR "Internal Server Error")

-- Trace processes spawned by Apache
index=windows_sysmon ParentImage="*httpd.exe"

-- Find post-exploitation recon
index=windows_sysmon *cmd.exe* *whoami*

-- Detect encoded PowerShell execution
index=windows_sysmon Image="*powershell.exe" (CommandLine="*enc*" OR CommandLine="*-EncodedCommand*")
```

**IOC checklist:**

| Layer | Indicator |
|---|---|
| Web | Long HTTP requests with Base64 strings in query params |
| Web | `/cgi-bin/` endpoints receiving `cmd=` parameters |
| Web | HTTP 500 errors on CGI scripts |
| Process | `httpd.exe` → `cmd.exe` parent-child relationship |
| Process | `httpd.exe` → `powershell.exe` execution |
| Post-exploitation | `whoami`, `ipconfig`, `net user` in Sysmon command lines |
| Post-exploitation | Base64-encoded PowerShell arguments |

**Sysmon field reference:**

| Field | Contains |
|---|---|
| `Image` | Full path of the process that ran |
| `ParentImage` | Full path of the process that launched it |
| `CommandLine` | Exact command executed, including all arguments |

**Blue Team recommendations:**
- Validate and sanitize all CGI script inputs
- Disable unnecessary CGI handlers
- Alert on web server processes spawning `cmd.exe` or `powershell.exe`
- WAF rule to flag Base64-encoded HTTP parameters
- Regular vulnerability scanning on web-facing endpoints
