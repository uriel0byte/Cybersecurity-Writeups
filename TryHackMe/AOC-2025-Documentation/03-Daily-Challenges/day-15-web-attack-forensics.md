# Day 15: Web Attack Forensics — Drone Alone

**Date:** December 15, 2025  
**Time Spent:** 1.5 hours  
**Difficulty:** ★★★★ *(Official rating: Easy — rated harder; query crafting from scratch and multi-source log correlation were both new)*  
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

### What is Sysmon?

System Monitor (Sysmon) is a Windows system service and driver from Microsoft's
Sysinternals suite. Once installed, it logs detailed telemetry about process
creation, network connections, file creation, and registry modifications directly
into the Windows Event Log.

The key thing Sysmon captures that standard Windows logging misses: **parent-child
process relationships**. It records not just that `cmd.exe` ran, but exactly which
process launched it, with the full command line arguments. This makes it essential
for detecting exploitation — a web server spawning a command shell is immediately
visible in Sysmon, whereas standard logs might only show that cmd.exe ran.

In this room, Sysmon logs were ingested into Splunk and stored in the
`windows_sysmon` index.

### What is a CGI Script?

CGI (Common Gateway Interface) is a standard that allows a web server to execute
programs and return their output as an HTTP response. In this room, `hello.bat` is
a CGI script — when a browser hits `/cgi-bin/hello.bat`, Apache runs the batch file
and sends back whatever it outputs.

The vulnerability: `hello.bat` takes a `cmd` parameter from the URL and passes it
directly to the system shell without any validation. An attacker can substitute any
command they want. The server executes it. That is command injection.

### Investigation Workflow

The core methodology for web attack forensics. This order matters — each step
answers a specific question before the next one makes sense.

| Step | Log Source | Index | Question |
|---|---|---|---|
| 1 | Apache Access Logs | `windows_apache_access` | Did the attack reach the server? |
| 2 | Apache Error Logs | `windows_apache_error` | Did the server try to execute it? |
| 3 | Sysmon Process Logs | `windows_sysmon` | Did the OS actually run something? |
| 4 | Sysmon Command History | `windows_sysmon` | What did the attacker do after? |
| 5 | Base64 Decode | External tool | What was the actual payload? |

### Step 1 — Detect Web Command Injection (Access Logs)

```spl
index=windows_apache_access (cmd.exe OR powershell OR "powershell.exe" OR "Invoke-Expression")
| table _time host clientip uri_path uri_query status
```

Searches Apache access logs for requests containing command execution strings.
The attacker's payload hides inside `uri_query` — the part of the URL after `?`.

**Finding:** Requests to `/cgi-bin/hello.bat?cmd=powershell` with long encoded
strings packed into the query parameter.

**Example encoded payload found in uri_query:**
```
VABoAGkAcwAgAGkAcwAgAG4AbwB3ACAATQBpAG4AZQAhACAATQBVAEEASABBAEEASABBAEEA
```

**Decoded:** `This is now Mine! MUAHAHAA`

**Decoding tool:** https://www.base64decode.org/

**Why UTF-16LE matters:** PowerShell's `-EncodedCommand` flag does not use standard
UTF-8 Base64. It uses **UTF-16LE** (little-endian Unicode) encoding. If you paste
a PowerShell-encoded string into a standard Base64 decoder and get garbage, this
is why — you need a decoder that handles UTF-16LE, or the output will be unreadable.
The string in this room decodes correctly because the decoder handles it properly.

**Why attackers encode:** Encoded strings look like random noise and won't match
any known malicious keyword patterns. Signature-based detection — tools that look
for strings like `Invoke-Mimikatz` or `DownloadString` — cannot detect what it
cannot read.

### Step 2 — Confirm Backend Execution (Error Logs)

```spl
index=windows_apache_error ("cmd.exe" OR "powershell" OR "Internal Server Error")
```

**View setting:** Switch to `Raw` in the event display dropdown above the results.

**Why HTTP 500 is significant here:** A 500 Internal Server Error on a CGI endpoint
that received a PowerShell command means the server passed the malicious input to
the system shell and something failed during execution. This is the dividing line
between a request that was received and a request that the backend actually acted
on. Step 1 proves the attack arrived. Step 2 proves it penetrated the web layer.

### Step 3 — Verify OS-Level Code Execution (Sysmon)

```spl
index=windows_sysmon ParentImage="*httpd.exe"
```

**View setting:** Switch to `Table` in the event display dropdown.

**Normal behavior:** `httpd.exe` (Apache) should only ever spawn its own worker
processes to handle web requests. It has no legitimate reason to launch system
utilities.

**What parent-child relationships prove:** When Sysmon shows `httpd.exe` as the
parent of `cmd.exe`, it means Apache directly launched the Windows command shell.
The only way that happens is if code running inside Apache told it to. That is
code execution confirmation — not a detection attempt, not a failed probe, actual
execution on the host operating system.

**Malicious finding:**
```
ParentImage: C:\Apache24\bin\httpd.exe
Image:       C:\Windows\System32\cmd.exe
```

Incident severity is HIGH/CRITICAL from this point. The attack moved from the
web layer into the OS.

### Step 4 — Track Post-Exploitation Reconnaissance

```spl
index=windows_sysmon *cmd.exe* *whoami*
```

**Why `whoami` is always first:** After gaining code execution on an unknown system,
an attacker's immediate question is "who am I running as?" The answer determines
everything — what files they can access, what commands they can run, whether they
need to escalate privileges before doing anything else.

| Result | Meaning |
|---|---|
| `NT AUTHORITY\SYSTEM` | Full control — highest possible privilege |
| `IIS APPPOOL\DefaultAppPool` | Web service account — limited but pivotable |
| Standard user | Restricted foothold — needs privilege escalation |

Finding `whoami` in Sysmon confirms the attack fully succeeded. The attacker is
no longer probing — they are actively operating inside the system.

**Kill chain at this point:**
```
Command Injection → Code Execution → OS Recon → [Privilege Esc / Lateral Movement / Persistence]
```

### Step 5 — Hunt Encoded PowerShell Payloads

```spl
index=windows_sysmon Image="*powershell.exe" (CommandLine="*enc*" OR CommandLine="*-EncodedCommand*" OR CommandLine="*Base64*")
```

Searches Sysmon for PowerShell executions where the command line contains encoding
flags. The `-EncodedCommand` (or shorthand `-enc`) parameter is how PowerShell
accepts and runs a Base64-encoded command directly from the command line, without
ever writing a script file to disk. This is a common technique for fileless
execution — the payload lives only in memory and in the command line argument.

**This room's result: no results.** The encoded payload was visible in the HTTP
request (Step 1), but PowerShell with an encoded command never actually executed
on the host OS. The attempt was made — the execution was not completed. That is
the correct Blue Team outcome.

**If this query returns results in other investigations:** decode every Base64
string found and document it. Real-world encoded payloads are reverse shells,
credential dumpers, persistence scripts, or ransomware droppers. The encoding
strips off in one step — the underlying command is usually immediately readable.

### Command Injection: Full Breakdown

**Vulnerable endpoint:** `/cgi-bin/hello.bat?cmd=<input>`

```
Normal request:
/cgi-bin/hello.bat?name=user
→ hello.bat processes "name=user"
→ Outputs: Hello, user!

Malicious request:
/cgi-bin/hello.bat?cmd=powershell -enc <BASE64>
→ hello.bat passes cmd value to system shell
→ Server executes: powershell -enc <BASE64>
→ Attacker has code execution
```

**Root cause:** The script passes user-supplied input directly to the system shell
with no validation, sanitization, or command whitelisting. It trusted whatever
arrived in the URL parameter.

**Why CGI scripts are particularly dangerous for this:** CGI scripts run with the
permissions of the web server process. On a Windows system, that can be a service
account with significant privileges. Every command the attacker injects runs as
that account.

---

## Challenges

The investigation logic and reading results clicked immediately — following the
evidence from web layer to OS layer made sense as a methodology. What didn't click:
writing queries independently. This room provided every query. A real SOC
investigation will not.

The gap is specific: Sysmon field names (`Image`, `ParentImage`, `CommandLine`) and
knowing which index holds which log type without being told. That is purely a reps
problem. The fix is more cold labs where the queries have to be built from scratch.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Command injection,
web application attacks, obfuscation techniques, fileless malware, reconnaissance.

**Domain 4.0 - Security Operations (28%):** SIEM operations, log analysis, incident
response, forensic investigation, threat hunting, alert triage.

---

## Evidence

![Splunk Command Injection Detection](../07-Screenshots/Day15-1.png)
*Command injection attempts in Apache access logs — Base64-encoded payload visible
in the uri_query field of requests hitting /cgi-bin/hello.bat.*

![Sysmon Process Tree Analysis](../07-Screenshots/Day15-2.png)
*httpd.exe spawning cmd.exe in Sysmon — parent-child relationship confirming
successful OS-level code execution.*

![Base64 Payload Decoding](../07-Screenshots/Day15-3.png)
*Decoded payload: "This is now Mine! MUAHAHAA" — attacker message confirming
full execution of the injected command.*

---

## Key Takeaways

**Investigation workflow:**
```
Web Access Logs → Error Logs → Sysmon Process Logs → Recon Commands → Payload Decoding
```

**Splunk query reference:**
```spl
-- Detect command injection in web requests
index=windows_apache_access (cmd.exe OR powershell OR "Invoke-Expression")
| table _time host clientip uri_path uri_query status

-- Confirm backend execution via error logs
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
| Web | HTTP 500 errors on CGI scripts following injection attempts |
| Process | `httpd.exe` → `cmd.exe` parent-child relationship |
| Process | `httpd.exe` → `powershell.exe` execution |
| Post-exploitation | `whoami`, `ipconfig`, `net user` in Sysmon command lines |
| Post-exploitation | `-enc` or `-EncodedCommand` flags in PowerShell command lines |

**Sysmon field reference:**

| Field | Contains |
|---|---|
| `Image` | Full path of the process that executed |
| `ParentImage` | Full path of the process that launched it |
| `CommandLine` | Exact command executed including all arguments |

**Blue Team recommendations:**
- Validate and sanitize all CGI script inputs — never pass raw user input to shell
- Disable CGI handlers not actively required
- Alert on web server processes spawning `cmd.exe` or `powershell.exe`
- WAF rule to flag Base64-encoded strings in HTTP query parameters
- Regular vulnerability scanning on web-facing endpoints

**Key terms:**

| Term | Definition |
|---|---|
| CGI (Common Gateway Interface) | Standard allowing a web server to execute programs and return output as HTTP |
| Command injection | Attack where unsanitized user input is passed to and executed by the system shell |
| Sysmon | Windows Sysinternals tool logging detailed process, network, and file activity including parent-child relationships |
| ParentImage | Sysmon field identifying which process spawned the current process |
| `-EncodedCommand` | PowerShell flag accepting a UTF-16LE Base64-encoded command string for execution |
| UTF-16LE | Little-endian Unicode encoding used by PowerShell's `-EncodedCommand`; standard Base64 decoders may not handle it correctly |
| Fileless execution | Running code entirely in memory without writing a script file to disk |
| uri_query | The portion of a URL after `?` containing query parameters — common hiding place for injection payloads |
| HTTP 500 | Internal Server Error; in CGI context often indicates the server attempted to execute malicious input |
