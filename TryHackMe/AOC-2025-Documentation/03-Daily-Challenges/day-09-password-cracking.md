# Day 9: Password Cracking — A Cracking Christmas

**Date:** December 9, 2025
**Time Spent:** 1 hour
**Difficulty:** ★☆☆☆
**Category:** Password Security / Cryptography
**Room:** https://tryhackme.com/room/attacks-on-ecrypted-files-aoc2025-asdfghj123

---

## Overview

Sir Carrotbane discovered encrypted PDF and ZIP files labeled "North Pole Asset List."
The task was to crack both using dictionary attacks with pdfcrack and John the Ripper.
Most of the cracking methodology was review. The genuinely new section covered endpoint
detection of password cracking — which matters for SOC work because offline cracking
bypasses every login-based alert you have.

---

## What I Learned

### Password-Based Encryption

Password-based encryption protects confidentiality. It does not prevent an attacker
from taking the file offline and guessing the password at their own pace with no
lockouts, no alerts, no failed login logs. Protection strength is entirely dependent
on password quality.

Different file formats use different encryption schemes. Older ZIP implementations
are notorious for weak encryption (ZipCrypto), which is why format matters when
assessing risk.

### Attack Methods

**Dictionary attack:** Run a wordlist against the file. Fast and effective against
weak or common passwords. `rockyou.txt` covers ~14.3 million real-world leaked
passwords from the 2009 RockYou breach.

**Brute-force/mask attack:** Try every combination within a defined character space.
Slow without GPU acceleration. Mask attacks narrow the search when you know the
password structure — `?l?l?l?d?d` means 3 lowercase letters followed by 2 digits.

Attacker order of operations:
```
1. Wordlist attack         (fast, low effort)
2. Targeted wordlists      (company names, personal info)
3. Mask attacks            (known password structure)
4. Full GPU brute-force    (last resort)
```

### Tools

**PDF cracking — pdfcrack:**
```bash
pdfcrack -f flag.pdf -w /usr/share/wordlists/rockyou.txt
```
Output shows words/sec, current word being tested, and prints
`found user-password: 'XXXXXXXXXX'` on success. Speed is hardware-dependent;
CPU-only on a VM typically runs tens of thousands of words/sec.

**ZIP cracking — John the Ripper:**

zip2john extracts the ZIP's password hash into a format John can read.
You cannot feed an encrypted ZIP directly to John.

```bash
# Step 1: Extract hash
zip2john flag.zip > ziphash.txt

# Step 2: Crack
john --wordlist=/usr/share/wordlists/rockyou.txt ziphash.txt

# Step 3: View result
john --show ziphash.txt
```

**File type verification:**
```bash
file flag.pdf
file flag.zip
```

### Detection of Password Cracking

Offline cracking produces no failed authentication events, no account lockouts,
no SIEM alerts from the identity provider. Detection has to happen on the endpoint
where the cracking is actually running.

**Process creation** is the strongest signal. On Windows, Sysmon Event ID 1 captures
full command lines. On Linux, auditd with execve syscall monitoring or an EDR sensor.

Binaries to flag:
```
john, hashcat, fcrackzip, pdfcrack, zip2john, pdf2john.pl, 7z, qpdf
```

Command-line patterns that indicate cracking:
```
--wordlist, -w, --rules, --mask, -a 3, -m
rockyou.txt, SecLists, zip2john, pdf2john
```

Artifact files cracking tools leave behind:
```
~/.john/john.pot           (John's cracked password store)
~/.hashcat/hashcat.potfile (Hashcat's equivalent)
~/.john/john.rec           (John session recovery file)
```

**GPU usage** is loud. Sustained high GPU utilization, fan curve spikes, and steady
power draw during off-hours are all anomalous. `nvidia-smi` shows long-running
hashcat or john processes against the GPU.

Libraries that indicate GPU cracking:
```
Windows: nvcuda.dll, OpenCL.dll, amdocl64.dll
Linux:   libcuda.so
```

**Network activity** precedes offline cracking. Watch for large wordlist downloads
(rockyou.txt is ~134 MB uncompressed), git clones of wordlist repositories like
SecLists, and package installs such as `apt install john hashcat`.

**File access patterns** — repeated reads of the same wordlist or encrypted file in
quick succession are unusual for normal user behavior and consistent with a cracking
loop.

### Detection Rules

**Sysmon (Windows):**
```
ProcessName = "C:\Program Files\john\john.exe" OR
ProcessName = "C:\Tools\hashcat\hashcat.exe" OR
CommandLine = "*pdf2john.pl*" OR
CommandLine = "*zip2john*"
```

**Linux auditd:**
```bash
auditctl -w /usr/share/wordlists/rockyou.txt -p r -k wordlists_read
auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/john -k crack_exec
```

Flag breakdown:
```
-w          Watch this file or directory
-p r        Trigger on read access
-k          Tag events with this key (searchable in logs)
-a          Append a rule
always      Evaluate on every syscall
exit        Log on syscall exit
-F          Filter field (arch=b64 = 64-bit, exe= = specific binary)
-S execve   Monitor process execution syscall
```

### Incident Response Playbook

```
1. Isolate host (malicious) OR tag/suppress alert (authorized pen test or lab)
2. Capture triage artifacts:
   - Running process list
   - Memory dump
   - nvidia-smi output
   - Open file handles
3. Preserve evidence:
   - Working directory contents
   - Wordlist files present
   - Hash files
   - Shell history
4. Assess impact:
   - Which files were targeted and successfully decrypted?
   - Any lateral movement post-cracking?
   - Exfiltration evidence?
5. Determine intent:
   - Authorized red team activity?
   - Escalate to full IR if confirmed malicious
6. Remediate:
   - Rotate all affected passwords
   - Enforce MFA where missing
7. Close with user education if insider or accidental
```

---

## Challenges

Cracking methodology was mostly review — dictionary attacks, rockyou.txt, pdfcrack,
and the zip2john workflow were all familiar. The detection section was the new
material. The concepts are clear but the specific auditd flag syntax and Sysmon
filter logic need more hands-on time before they are fully solid. That gap is worth
closing during log analysis labs.

---

## Security+ Alignment

**Domain 1.0 - General Security Concepts (12%):** Password policies, authentication
security, cryptography, password hashing.

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Password attacks,
brute-force attacks, credential compromise.

**Domain 4.0 - Security Operations (28%):** Incident detection, monitoring and
logging, incident response.

---

## Key Takeaways

**Commands:**
```bash
# Identify file type
file flag.pdf
file flag.zip

# PDF — dictionary attack
pdfcrack -f flag.pdf -w /usr/share/wordlists/rockyou.txt

# ZIP — extract hash, then crack
zip2john flag.zip > ziphash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt ziphash.txt
john --show ziphash.txt
```

**Detection signals:**

| Signal | Indicators | Where to Look |
|---|---|---|
| Process | john, hashcat, pdfcrack, zip2john execution | Sysmon EID 1, auditd, EDR |
| GPU | Sustained high utilization, power draw | nvidia-smi, GPU monitoring |
| Network | rockyou.txt download, SecLists clone, tool installs | Proxy logs, EDR telemetry |
| File access | Repeated wordlist reads, repeated encrypted file reads | auditd, file access logs |

**Command-line red flags:**
```
--wordlist / -w rockyou.txt
--mask / -a 3
zip2john / pdf2john
```

**Key terms:**

| Term | Definition |
|---|---|
| Dictionary attack | Password guessing using a predefined wordlist |
| Brute-force | Exhaustive search of all possible character combinations |
| Mask attack | Brute-force with a constrained character space (e.g., `?l?l?l?d?d`) |
| zip2john | Extracts a crackable hash from an encrypted ZIP |
| rockyou.txt | ~14.3 million real passwords from the 2009 RockYou breach |
| john.pot | John the Ripper's file storing previously cracked passwords |
| Sysmon EID 1 | Windows process creation event — captures full command line |
| execve | Linux syscall for executing a program — monitored by auditd |
| ZipCrypto | Legacy weak ZIP encryption; AES-256 is the modern secure alternative |
