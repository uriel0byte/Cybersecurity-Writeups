# Day 13: YARA Rules — YARA Mean One!

**Date:** December 13, 2025  
**Time Spent:** 2 hours  
**Difficulty:** ★★★☆  
**Category:** Malware Detection / Threat Hunting / Digital Forensics  
**Room:** https://tryhackme.com/room/yara-aoc2025-q9w1e3y5u7

---

## Overview

McSkidy sent encrypted images with hidden messages that could only be decoded
using custom YARA rules. The task was to write a rule using regex to scan
for the pattern `TBFC:` followed by alphanumeric codewords across a directory
of image files. YARA was completely new — no prior exposure from Security+
or anywhere else.

---

## What I Learned

### What is YARA?

YARA is a pattern-matching tool used to identify and classify malware. Instead
of relying on file names or antivirus signatures, it scans files and memory
for specific patterns — strings, byte sequences, or regex — that indicate
malicious behavior. Detection is based on what the file contains and does,
not what it's called.

**Why this matters for SOC work:** YARA lets you move from passive monitoring
to active hunting. Once you have an IOC — a string, a hex pattern, a URL
format — you can turn it into a rule and scan your entire environment for it
immediately, without waiting for an AV vendor to push an update.

### When Defenders Use YARA

**Post-incident sweeps** — Malware found on one host. Does it exist anywhere
else on the network? Write a rule from the sample and scan everything.

**Proactive threat hunting** — Search endpoints and file shares for known
malware family signatures before they trigger alerts.

**Intelligence-driven scans** — Apply YARA rules from threat intelligence
feeds or community sources (VirusTotal, GitHub) to existing environments.

**Memory analysis** — Scan RAM dumps for fileless malware that never touches
disk and would be invisible to file-based scanning.

**Large-scale file scanning** — Identify infected files across thousands of
directories without opening each one.

### YARA Rule Structure

Every rule has three sections:

```
meta      → Information about the rule (author, date, purpose)
strings   → The patterns YARA searches for
condition → The logic that decides when the rule fires
```

**Basic structure:**
```yara
rule RuleName
{
    meta:
        author      = "Analyst Name"
        description = "What this detects"
        date        = "2025-10-10"

    strings:
        $s1   = "rundll32.exe" fullword ascii
        $s2   = "msvcrt.dll" fullword wide
        $url1 = /http:\/\/.*malhare.*/ nocase

    condition:
        any of them
}
```

### Strings — Types and Modifiers

**Three string types:**

**1. Text strings** — Readable characters. Default is ASCII, case-sensitive.
```yara
$text = "Christmas"
```

**2. Hexadecimal strings** — Raw bytes. Use when there are no readable strings.
```yara
$mz  = { 4D 5A }           // MZ header — Windows PE file signature
$hex = { 48 8B ?? ?? 48 89 } // ?? = wildcard, matches any single byte
```

Wildcard options:
```
??      any single byte
[2-4]   any 2 to 4 bytes
```

**3. Regular expressions** — Flexible patterns for variable content.
```yara
$url = /http:\/\/.*malhare.*/ nocase
$cmd = /powershell.*-enc\s+[A-Za-z0-9+\/=]+/ nocase
```

Regex is powerful but can slow scans if written too broadly.

---

**String modifiers** (append after the string value):

| Modifier | What It Does |
|---|---|
| `nocase` | Case-insensitive matching |
| `ascii` | Match ASCII (1-byte) encoding |
| `wide` | Match Unicode (2-byte) encoding — common in Windows executables |
| `fullword` | Match only if surrounded by non-alphanumeric characters |
| `xor` | Test all 256 single-byte XOR keys 0x00–0xFF (detects XOR-obfuscated strings) |
| `base64` | Decode Base64 content and match original pattern |
| `base64wide` | Same as base64 but for Unicode-encoded Base64 |

Modifiers can be combined: `"Christmas" wide ascii nocase`

### Conditions

The condition decides when the rule fires. It uses the string variables
defined in the strings section combined with logical operators.

```yara
// Single string
condition: $xmas

// Any one of the defined strings
condition: any of them

// All defined strings must be present
condition: all of them

// Logic operators — and / or / not
condition: ($s1 or $s2) and not $benign

// File size constraint
condition: any of them and filesize < 700KB
```

File size constraint is useful because malware loaders tend to be small.
Legitimate software is usually larger, so combining a string match with
`filesize < Xmb` reduces false positives.

### IcedID Rule — Practical Example

```yara
rule TBFC_Simple_MZ_Detect
{
    meta:
        author      = "TBFC SOC L2"
        description = "IcedID Rule"
        date        = "2025-10-10"
        confidence  = "low"

    strings:
        $mz   = { 4D 5A }                // MZ header (Windows PE)
        $hex1 = { 48 8B ?? ?? 48 89 }    // Malicious binary fragment
        $s1   = "malhare" nocase         // IOC string

    condition:
        all of them and filesize < 10485760  // All present + under 10MB
}
```

`{ 4D 5A }` is the ASCII hex for "MZ" — the magic bytes at the start of
every Windows executable (PE file). Finding this plus the other strings
narrows matches to suspicious executables only.

### Running YARA

```bash
# Basic scan (single file or directory)
yara rule.yar /path/to/scan

# Recursive — scan all subdirectories
yara -r rule.yar /path/to/scan

# Recursive + show matched strings
yara -r -s rule.yar /path/to/scan
```

**Example output:**
```
TBFC_Simple_MZ_Detect C:\Users\WarevilleElf\AppData\Roaming\malhare_gift_loader.exe
```

### Practical Task — Decoding McSkidy's Message

The task was to scan `/home/ubuntu/Downloads/easter` for the pattern
`TBFC:` followed by alphanumeric codewords hidden inside image files.

**Regex pattern used:**
```
/TBFC:[A-Za-z0-9]+/
```

Breaking it down:
```
TBFC:        literal string match
[A-Za-z0-9]  character class — any uppercase letter, lowercase letter, or digit
+            one or more of the preceding character class
```

**Rule written for the task:**
```yara
rule Find_TBFC_Codewords
{
    meta:
        description = "Extract TBFC codewords from image files"

    strings:
        $codeword = /TBFC:[A-Za-z0-9]+/

    condition:
        $codeword
}
```

Run with `-s` to print the matched strings and read the codewords in order.

---

## Challenges

YARA was completely new. The structure feels like writing code — defining
variables, applying modifiers, combining logic — which was uncomfortable
without a programming background. The basic concept clicked quickly (scan
for patterns, fire on match) but regex syntax and hex patterns required
multiple re-reads. Knowing which modifiers to use and when (`wide`? `ascii`?
both?) only became clearer after seeing the examples side by side. This is
a skill that will sharpen with practice, not reading.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Malware
indicators, threat intelligence, IOC identification, signature-based detection.

**Domain 4.0 - Security Operations (28%):** Threat hunting, incident response,
malware analysis, forensics, detection tools and techniques.

---

## Evidence

![Regex Pattern Matching](../07-Screenshots/Day13-1.png)
*YARA rule with regex pattern `/TBFC:[A-Za-z0-9]+/` scanning image files
for hidden codewords.*

![YARA Scan Results](../07-Screenshots/Day13-2.png)
*Recursive YARA scan results showing matched codewords extracted from the
easter directory.*

---

## Key Takeaways

**Rule template:**
```yara
rule RuleName
{
    meta:
        description = "What it detects"
        author      = "Name"
        date        = "YYYY-MM-DD"

    strings:
        $text  = "suspicious string" nocase
        $hex   = { 4D 5A }
        $regex = /pattern.*here/ nocase

    condition:
        any of them
}
```

**String modifiers quick reference:**
```
nocase       case-insensitive
ascii        1-byte encoding
wide         2-byte Unicode encoding
fullword     not part of a larger word
xor          test all 256 single-byte XOR keys (0x00–0xFF)
base64       match after Base64 decode
base64wide   match after Base64 decode (Unicode)
```

**Condition patterns:**
```yara
$string                          single match
any of them                      at least one match
all of them                      every string present
($s1 or $s2) and not $benign     logic operators
any of them and filesize < 1MB   with file size check
```

**CLI flags:**
```bash
yara rule.yar /path           basic scan
yara -r rule.yar /path        recursive
yara -r -s rule.yar /path     recursive + print matched strings
```

**Common hex signatures:**
```
4D 5A              MZ — Windows PE executable header
50 4B 03 04        PK — ZIP archive
25 50 44 46        %PDF — PDF file
FF D8 FF           JPEG image header
```

**Key terms:**

| Term | Definition |
|---|---|
| YARA | Pattern-matching tool for identifying and classifying malware |
| IOC | Indicator of Compromise — artifact proving a system was attacked |
| MZ header | `4D 5A` magic bytes marking the start of a Windows PE file |
| `nocase` | Modifier making string matching case-insensitive |
| `wide` | Modifier matching 2-byte Unicode encoding |
| `xor` | Modifier testing all 256 single-byte XOR keys (0x00–0xFF) |
| `fullword` | Modifier requiring string to be a standalone word |
| Fileless malware | Malware running only in RAM, never written to disk |
| `any of them` | Condition firing if at least one defined string matches |
| `all of them` | Condition firing only if every defined string matches |
