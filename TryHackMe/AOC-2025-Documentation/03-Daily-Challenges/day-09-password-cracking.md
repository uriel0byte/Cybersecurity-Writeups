# Day 9: Password Cracking — A Cracking Christmas

**Date:** December 9, 2025 | **Time:** ~1 hour | **Difficulty:** Easy
**Category:** Password Security / Cryptography
**Room:** https://tryhackme.com/room/attacks-on-ecrypted-files-aoc2025-asdfghj123

Note: "ecrypted" in the URL appears to be a typo in the original room link — likely "encrypted."

---

## What the Room Was About

Sir Carrotbane found encrypted PDF and ZIP files labeled "North Pole Asset List." The task was to crack both using dictionary attacks. Most of the cracking methodology was review. The genuinely new section covered **endpoint detection of password cracking** — which matters for SOC work because offline cracking bypasses every login-based alert you have.

---

## Core Concepts

### Why Encryption Alone Isn't Enough

Password-based encryption protects confidentiality. It does not prevent an attacker from taking the file offline and guessing the password at their own pace with no lockouts, no alerts, no failed login logs. Protection strength is entirely dependent on password quality.

Different file formats use different encryption schemes. Older ZIP implementations in particular are notorious for weak encryption (ZipCrypto), which is why format matters when assessing risk.

---

### Attack Methods

**Dictionary attack:** Run a wordlist against the file. Fast, effective against weak or common passwords. `rockyou.txt` alone covers ~14.3 million real-world leaked passwords.

**Brute-force/mask attack:** Try every combination within a defined character space. Slow unless GPU-accelerated. Mask attacks narrow the search space when you know the password structure (e.g., `?l?l?l?d?d` = 3 lowercase letters + 2 digits).

Attacker order of operations:
```
1. Wordlist attack (fast, low effort)
2. Targeted wordlists (company/project names, personal info)
3. Mask attacks (known structure)
4. Full GPU-accelerated brute-force
```

---

## Tools and Commands

### PDF Cracking — pdfcrack

```bash
pdfcrack -f flag.pdf -w /usr/share/wordlists/rockyou.txt
```

Output shows words/sec, current word being tested, and prints `found user-password: 'XXXXXXXXXX'` on success. Speed is hardware-dependent; CPU-only cracking on a VM typically runs in the range of tens of thousands of words/sec.

---

### ZIP Cracking — John the Ripper

zip2john extracts the ZIP's password hash into a format John can work with. You cannot feed an encrypted ZIP directly to John.

```bash
# Step 1: Extract hash
zip2john flag.zip > ziphash.txt

# Step 2: Crack
john --wordlist=/usr/share/wordlists/rockyou.txt ziphash.txt

# Step 3: View result
john --show ziphash.txt
```

---

### File Type Verification

```bash
file flag.pdf
file flag.zip
```

Always confirm what you're actually dealing with before reaching for a cracking tool.

---

## Detection of Password Cracking (SOC — New Material)

This is the section worth focusing on. Offline cracking produces no failed authentication events, no account lockouts, no SIEM alerts from your identity provider. Detection has to happen on the endpoint where the cracking is running.

---

### Signal 1: Process Creation

Watch for cracking tool execution. On Windows, Sysmon Event ID 1 captures full command lines. On Linux, auditd with execve syscall monitoring or an EDR sensor.

**Binaries to flag:**
```
john, hashcat, fcrackzip, pdfcrack, zip2john, pdf2john.pl, 7z, qpdf
```

**Command-line patterns that indicate cracking:**
```
--wordlist, -w, --rules, --mask, -a 3, -m
rockyou.txt, SecLists, zip2john, pdf2john
```

**Artifact files cracking tools leave behind:**
```
~/.john/john.pot          (John's cracked password store)
~/.hashcat/hashcat.potfile (Hashcat's equivalent)
john.rec                  (session recovery file)
```

---

### Signal 2: GPU and Resource Usage

GPU cracking is loud. A legitimate user does not pin their GPU at 99% for 20 minutes at 11 PM.

- Sustained high GPU utilization visible in `nvidia-smi`
- Fan curve spike corresponding to compute load
- High, steady power draw

**Libraries that indicate GPU cracking is running:**
```
Windows: nvcuda.dll, OpenCL.dll, amdocl64.dll
Linux:   libcuda.so
```

---

### Signal 3: Network Activity

Once wordlists are present, cracking is fully offline. But before that point:

- Large file downloads (rockyou.txt is ~134 MB uncompressed)
- Git clones of wordlist repositories (SecLists, etc.)
- Package installs: `apt install john hashcat`
- GPU driver downloads

---

### Signal 4: File Access Patterns

Repeated reads of the same wordlist or encrypted file in quick succession — unusual for normal user behavior, consistent with a cracking loop.

---

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

Flag breakdown (for review):
```
-w        Watch this file or directory
-p r      Trigger on read access
-k        Tag events with this key (searchable in logs)
-a        Append a rule
always    Evaluate rule on every syscall
exit      Log on syscall exit
-F        Filter (arch=b64 = 64-bit syscalls, exe= = specific binary)
-S execve Monitor process execution syscall
```

Sysmon filter syntax and Sigma rule structure are still being learned — flag for follow-up.

---

### Incident Response Playbook

When cracking activity is detected:

```
1. Isolate host (if malicious) OR tag/suppress alert (if authorized lab/pen test)
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
   - Authorized red team / pen test activity?
   - Escalate to full IR if confirmed malicious
6. Remediate:
   - Rotate all affected passwords
   - Enforce MFA where missing
7. Close with user education if insider/accidental
```

---

## Challenges

Cracking methodology was review. The detection section was new. The auditd flag syntax and Sysmon filter logic are partially understood — the concepts are clear, but the specific syntax requires more time with both tools before it's solid. That's a gap worth closing when working through log analysis labs.

---

## Security+ Mapping

**Domain 1.0 — General Security Concepts (12%):** Password policies, authentication security, cryptography, password hashing.

**Domain 2.0 — Threats, Vulnerabilities and Mitigations (22%):** Password attacks, brute-force attacks, credential compromise.

**Domain 4.0 — Security Operations (28%):** Incident detection, monitoring and logging, incident response.

---

## Quick Reference

### Commands

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

### Detection Summary

| Signal | Indicators | Where to Look |
|---|---|---|
| Process | john, hashcat, pdfcrack, zip2john execution | Sysmon EID 1, auditd, EDR |
| GPU | Sustained high utilization, power draw | nvidia-smi, GPU monitoring |
| Network | rockyou.txt download, SecLists clone, tool installs | Proxy logs, EDR telemetry |
| File access | Repeated wordlist reads, repeated encrypted file reads | auditd, file access logs |

### Command-Line Red Flags

```
--wordlist / -w rockyou.txt
--mask / -a 3
zip2john / pdf2john
```

### Key Terms

| Term | Definition |
|---|---|
| Dictionary attack | Password guessing using a predefined wordlist |
| Brute-force | Exhaustive search of all possible combinations |
| Mask attack | Brute-force with constrained character space (e.g., `?l?l?l?d?d`) |
| zip2john | Utility to extract a crackable hash from an encrypted ZIP |
| rockyou.txt | ~14.3 million real passwords from the 2009 RockYou breach |
| john.pot | John the Ripper's file storing previously cracked passwords |
| Sysmon EID 1 | Windows event for process creation — captures full command line |
| execve | Linux syscall for executing a program — monitored by auditd |
