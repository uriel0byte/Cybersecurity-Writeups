# Day 15: Web Attack Forensics — Drone Alone

**Date:** December 15, 2025  
**Time Spent:** 1.5 hours  
**Difficulty:** ★★★★  
**Category:** SIEM / Web Attack Forensics / Blue Team  
**Room:** https://tryhackme.com/room/webattackforensics-aoc2025-b4t7c1d5f8

---

## Overview

TBFC's drone scheduler web UI was sending strange, long HTTP requests
containing Base64 chunks. Splunk raised an alert: Apache spawned an unusual
process. The task was to triage the incident using Splunk by pivoting between
Apache access logs, Apache error logs, and Sysmon telemetry — identify the
attack vector, confirm exploitation, trace post-exploitation activity, decode
obfuscated payloads, and reconstruct the full chain. This is the type of
Blue Team investigation work the rest of the learning has been building
toward.

**Splunk login:** `Blue` / `Pass1234`. Set time range to All Time before
running any query or results will come back empty.

---

## What I Learned

### Investigation Strategy

The investigation follows a logical pivot chain across three log sources:

```
windows_apache_access  → detect the attack coming in
windows_apache_error   → confirm it reached the backend
windows_sysmon         → prove it executed at OS level
```

Each step answers a specific question before moving to the next. This is the
core SOC analyst workflow: detect → confirm → validate → investigate →
analyze.

---

### Step 1: Detect Suspicious Web Requests (Access Logs)

**Question:** Are there HTTP requests showing command execution attempts?

```spl
index=windows_apache_access (cmd.exe OR powershell OR "powershell.exe" OR "Invoke-Expression")
| table _time host clientip uri_path uri_query status
```

**What this finds:** Apache access logs filtered for strings associated with
command execution. The table shows timestamp, target host, attacker IP,
requested path, query parameters, and HTTP status.

**What was found:**
- Requests hitting `/cgi-bin/hello.bat?cmd=powershell`
- Query parameters containing long Base64-encoded strings

**Example payload in uri_query:**
```
VABoAGkAcwAgAGkAcwAgAG4AbwB3ACAATQBpAG4AZQAhACAATQBVAEEASABBAEEASABBAEEA
```

**Decoded result:** `This is now Mine! MUAHAHAA`

**Encoding note:** PowerShell's `-EncodedCommand` parameter uses Base64-encoded
**UTF-16LE** (Unicode), not plain ASCII. `base64decode.org` handles the
conversion automatically. This is why raw base64 output for this string shows
wide characters — every ASCII character is two bytes in UTF-16LE.

**Attack vector confirmed:** Command injection through the vulnerable CGI
script `hello.bat`.

---

### Step 2: Confirm Backend Execution (Error Logs)

**Question:** Did the malicious request reach the backend and get processed?

```spl
index=windows_apache_error ("cmd.exe" OR "powershell" OR "Internal Server Error")
```

**Important:** Set display to `View: Raw` to see full error content.

**What to look for:** HTTP 500 Internal Server Error on CGI endpoints.

A 500 on `/cgi-bin/hello.bat?cmd=powershell` means:
- The server passed the input to the backend
- The backend attempted to execute it
- Something failed during execution

This is the key distinction between an **attempt** (blocked at the web layer)
and **exploitation** (input processed by the server). A 500 here confirms
the attack penetrated beyond the web layer.

---

### Step 3: Trace Process Creation (Sysmon)

**Question:** Did Apache actually spawn system processes?

```spl
index=windows_sysmon ParentImage="*httpd.exe"
```

**Important:** Set display to `View: Table` to see process relationships
clearly.

**Normal Apache behaviour:** `httpd.exe` spawns only its own worker threads.

**What was found:**
```
ParentImage: C:\Apache24\bin\httpd.exe
Image:       C:\Windows\System32\cmd.exe
```

Apache spawning `cmd.exe` is the strongest single indicator of successful
command injection. The attack has now moved from the web layer into the
operating system. Incident severity: **Critical**.

---

### Step 4: Confirm Reconnaissance (Sysmon)

**Question:** What did the attacker do after gaining execution?

```spl
index=windows_sysmon *cmd.exe* *whoami*
```

**What was found:** `whoami.exe` executed via `cmd.exe`.

`whoami` is almost always the first command an attacker runs after gaining
code execution. It tells them which user account the injected process is
running as, which determines their privilege level and next steps.

Finding `whoami.exe` in Sysmon confirms:
- Code execution was fully successful
- The attacker is actively performing post-exploitation reconnaissance
- The attack chain is: **web injection → OS shell → recon**

---

### Step 5: Check for Encoded PowerShell Execution (Sysmon)

**Question:** Did any Base64-encoded PowerShell payloads actually execute?

```spl
index=windows_sysmon Image="*powershell.exe" (CommandLine="*enc*" OR CommandLine="*-EncodedCommand*" OR CommandLine="*Base64*")
```

**Result: No results returned.**

The attacker injected PowerShell via the web request, but the encoded payload
never successfully executed at the OS level. `cmd.exe` and `whoami.exe`
ran — but `powershell.exe` with encoded commands did not complete. This means
some defensive layer stopped the PowerShell stage specifically.

This is a critical distinction for the incident report: command injection
succeeded, recon ran, but the more dangerous encoded payload was blocked.

---

### Command Injection Explained

The vulnerability is in `hello.bat` — a CGI script that passes user input
directly to the system shell without validation.

```
Normal:    /cgi-bin/hello.bat?name=user
           → hello.bat processes "name" → outputs "Hello, user!"

Malicious: /cgi-bin/hello.bat?cmd=powershell -enc <BASE64>
           → hello.bat passes cmd parameter to shell
           → server executes: powershell -enc <BASE64>
           → attacker has code execution
```

**Root cause:** No input validation or sanitisation on the `cmd` parameter.
The script treats attacker-controlled input as a trusted system command.

---

### Base64 as Obfuscation

Base64 is not encryption. It is encoding — reversible by anyone with a
decoder. Attackers use it because:
- Encoded strings do not match plaintext signature rules
- Tools scanning for `powershell`, `whoami`, `net user` in HTTP traffic will
  miss Base64-wrapped versions
- It buys time against signature-based detection

The moment you see a long Base64 string in `uri_query` or a PowerShell
`-EncodedCommand` / `-enc` flag in Sysmon logs, decode it immediately.

**Decode tool:** https://www.base64decode.org

---

## Challenges

This type of investigation — SIEM, log correlation, decoding payloads,
reconstructing the chain — is the work I want to do. The methodology and
reasoning clicked immediately. The hard part is independent query crafting.
The room provided the queries, and following them was straightforward.
Writing them from scratch without a template, remembering field names like
`ParentImage`, `CommandLine`, `uri_query`, and knowing which index holds which
log type — that takes repeated practice to internalise. This is the gap to
close in future labs.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Command
injection, web application attacks, obfuscation techniques, reconnaissance.

**Domain 4.0 - Security Operations (28%):** SIEM operations, log analysis,
incident response, forensic investigation, threat hunting, alert triage.

---

## Evidence

![Splunk Command Injection Detection](../07-Screenshots/Day15-1.png)
*Apache access logs showing command injection attempts with Base64-encoded
payloads in uri_query parameters targeting /cgi-bin/hello.bat.*

![Sysmon Process Tree Analysis](../07-Screenshots/Day15-2.png)
*Sysmon confirming httpd.exe spawned cmd.exe — successful command injection
at OS level.*

![Base64 Payload Decoding](../07-Screenshots/Day15-3.png)
*Base64 payload decoded to "This is now Mine! MUAHAHAA" via base64decode.org.*

---

## Key Takeaways

**Investigation workflow:**
```
1. Access logs  → suspicious HTTP requests, Base64 in uri_query
2. Error logs   → HTTP 500 on CGI endpoint = reached backend
3. Sysmon       → httpd.exe → cmd.exe = OS-level execution confirmed
4. Sysmon       → cmd.exe → whoami.exe = post-exploitation recon
5. Sysmon       → powershell.exe -enc = check if payload executed
```

**Splunk queries:**

Step 1 — Detect web command injection:
```spl
index=windows_apache_access (cmd.exe OR powershell OR "powershell.exe" OR "Invoke-Expression")
| table _time host clientip uri_path uri_query status
```

Step 2 — Confirm backend execution:
```spl
index=windows_apache_error ("cmd.exe" OR "powershell" OR "Internal Server Error")
```

Step 3 — Trace process creation:
```spl
index=windows_sysmon ParentImage="*httpd.exe"
```

Step 4 — Find reconnaissance:
```spl
index=windows_sysmon *cmd.exe* *whoami*
```

Step 5 — Detect encoded PowerShell:
```spl
index=windows_sysmon Image="*powershell.exe" (CommandLine="*enc*" OR CommandLine="*-EncodedCommand*" OR CommandLine="*Base64*")
```

**Attack chain confirmed:**
```
/cgi-bin/hello.bat?cmd=powershell -enc <BASE64>
        ↓
HTTP 500 (backend processed input)
        ↓
httpd.exe → cmd.exe (OS execution)
        ↓
cmd.exe → whoami.exe (recon)
        ↓
powershell -enc → BLOCKED (no Sysmon events)
```

**IOCs to extract:**

| Layer | Indicator |
|---|---|
| Web | Long Base64 strings in uri_query |
| Web | `/cgi-bin/` requests with `cmd=` parameter |
| Web | HTTP 500 on CGI endpoints |
| Process | `httpd.exe` spawning `cmd.exe` or `powershell.exe` |
| Process | `whoami.exe` spawned by web server process tree |
| Command | `-enc` or `-EncodedCommand` in PowerShell arguments |

**Key terms:**

| Term | Definition |
|---|---|
| Command injection | Vulnerability allowing attacker to execute OS commands through app input |
| CGI | Common Gateway Interface — runs server-side scripts via web requests |
| `ParentImage` | Sysmon field showing which process spawned the current process |
| `CommandLine` | Sysmon field showing the full command used to launch a process |
| `-EncodedCommand` / `-enc` | PowerShell flag accepting Base64-encoded UTF-16LE commands |
| UTF-16LE | Character encoding used by PowerShell encoded commands (2 bytes per char) |
| `whoami.exe` | Windows binary returning current user context — first recon step |
| HTTP 500 | Internal Server Error — server processed input but execution failed |
