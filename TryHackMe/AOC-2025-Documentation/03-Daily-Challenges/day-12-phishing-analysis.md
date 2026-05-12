# Day 12: Phishing Analysis — Phishmas Greetings

**Date:** December 12, 2025  
**Time Spent:** 2 hours  
**Difficulty:** ★★★☆  
**Category:** Security Awareness / Phishing Analysis / Email Security  
**Room:** https://tryhackme.com/room/spottingphishing-aoc2025-r2g4f6s8l0

---

## Overview

TBFC's email protection platform went offline, forcing manual triage of every
suspicious message. Joined the Incident Response Task Force to classify emails
as spam, phishing, or legitimate — then identify three specific indicators per
phishing attempt. Most of the techniques here were new: email header analysis,
punycode, SPF/DKIM/DMARC, and modern legitimate-platform phishing flows. Security+
only covered surface-level phishing definitions.

---

## What I Learned

### Spam vs. Phishing

These are not the same threat level and should not be triaged the same way.

**Spam** is bulk, untargeted, low-precision noise. The goal is volume — push
advertising, run scams, harvest active email addresses. Low risk. Annoying.

**Phishing** is a precision strike. Carefully crafted to deceive a specific
target into handing over credentials, downloading malware, or transferring
money. High risk. Actual threat.

The distinction matters for triage: not every unwanted email is an incident.
The question is intention — what is this email trying to get the user to do?

---

### Impersonation

Attacker uses a trusted person's name in the display field while sending from
a completely different domain.

**Example:**
```
Display name: McSkidy
Sender email: mcskidy@gmail.com    ← free domain, not internal
```

A legitimate TBFC employee sends from `@tbfc.com`, not a Gmail address. Any
mismatch between display name and sender domain is an immediate red flag.

**What to check:**
- Does the sender domain match the company domain?
- Is it a free email service (gmail.com, yahoo.com, outlook.com)?
- Does the email structure match the company's standard format?

---

### Social Engineering

Manipulation of emotion rather than exploitation of a technical vulnerability.
The goal is to make the target act before they think.

**Common triggers:**
```
Fear      → "Your account will be suspended immediately"
Urgency   → "URGENT: Action required now"
Authority → Impersonating IT, management, or HR
Curiosity → "You won't believe this document"
```

**Indicators found in the McSkidy phishing email:**
- `URGENT` in subject line — pressure tactic
- Request for VPN credentials — no legitimate IT request asks for this
- Side channel redirect — "don't reply here, use this link instead"

Side channel redirect is a major red flag: any email that discourages using
standard communication channels is trying to move the conversation somewhere
your security team cannot monitor.

**Rule:** Legitimate urgent IT requests always allow verification through
official channels. If an email says "don't verify through normal means," it's
social engineering.

---

### Typosquatting and Punycode

Both techniques create domains that look legitimate to a human but are
completely different to a computer.

**Typosquatting:** Register a misspelled version of a legitimate domain.
```
github.com  →  glthub.com   (lowercase L replacing i)
tbfc.com    →  tbcc.com
```
Relies on users not reading carefully or making typing mistakes.

**Punycode:** An encoding system that converts Unicode (international)
characters to ASCII for use in domain names. Attackers use characters from
non-Latin alphabets that are visually identical to Latin letters.

```
tryhackme.com   ← legitimate (Latin characters)
тryhackme.com   ← fake (Cyrillic т replacing Latin t)
```

These look identical in email previews. The computer sees two completely
different domains.

**How to detect punycode in email headers:**

Check the `Return-Path` field. Punycode domains use the ACE prefix `xn--`:
```
Legitimate:  return-path: user@tbfc.com
Punycode:    return-path: user@xn--tbfc-xqa.com
```

**Challenge example:** Sender domain contained Latin `ƒ` (f with hook)
instead of a standard `f`. Looked legitimate in preview, different domain
entirely.

---

### Email Spoofing and Header Analysis

Spoofing makes an email appear to come from a legitimate domain while actually
originating elsewhere. The display name and `From:` field look real — headers
expose the truth.

**Key headers to check:**

**Authentication-Results** — Added by the receiving mail server. Shows
whether the email passed SPF, DKIM, and DMARC checks.

```
spf=fail      → Sending server not authorized for this domain
dkim=fail     → Signature missing or invalid
dmarc=fail    → Failed policy check based on SPF/DKIM results
```

SPF fail + DKIM fail = strong indicator of spoofing.

**What each protocol does:**

| Protocol | What It Checks |
|---|---|
| SPF | Whether the sending server is authorized to send for that domain (DNS record) |
| DKIM | Cryptographic signature proving the email wasn't altered in transit |
| DMARC | Policy layer — uses SPF + DKIM results to accept, quarantine, or reject |

**Return-Path** — Contains the actual envelope sender address used for
bounces. Set by the sending mail server, not the sender. If this differs
from the `From:` address, the `From:` was forged.

```
From:         mcskidy@tbfc.com       ← what user sees
Return-Path:  attacker@hopsec.thm    ← where bounces actually go
```

This mismatch confirms spoofing.

---

### Malicious Attachments

Classic phishing vector: attach a malicious file disguised as something
expected (invoice, voice message, document).

**Challenge example:** Email claimed to contain a voice message from McSkidy.
Attachment appeared to be audio. Actual file type: `.hta`.

**Why `.hta` files are dangerous:**
HTA (HTML Application) files are executed by `mshta.exe` (Microsoft HTML
Application Host), completely outside the browser sandbox. They have direct
access to the file system, registry, and network — effectively the same
privileges as the current user. Executing one hands the attacker a foothold.

**Malicious file types to know:**

| Type | Risk |
|---|---|
| `.hta` | Runs outside browser sandbox, full system access |
| `.html` | Can execute JavaScript, redirect, harvest credentials |
| `.exe` / `.bat` | Direct execution |
| `.js` / `.vbs` | Script execution via Windows Script Host |
| `.zip` / `.rar` | Archive concealing any of the above |
| `.docm` / `.xlsm` | Office documents with enabled macros |

Red flag combination: unexpected attachment + urgent/emotional subject line.

---

### Modern Phishing Trends

Email security improved enough that sending malware directly became
unreliable. Attackers adapted by using legitimate platforms as delivery
infrastructure — the link passes email filters because it points to a real
service.

**Modern phishing flow:**
```
Phishing email
      ↓
Link to legitimate platform (Dropbox, Google Drive, OneDrive, SharePoint)
      ↓
Fake shared document
      ↓
Fake login page (credential theft) OR malicious file download (malware)
```

**Why it works:**
- The email link points to a real, trusted domain — filters pass it
- User is now outside company security controls
- Familiar platform reduces suspicion

**Challenge example:**
```
Subject: "TBFC-IT shared 'Christmas Laptop Upgrade Agreement' with you"
```
Combined three techniques simultaneously:
- Punycode domain (trust manipulation)
- OneDrive link (legitimate platform, bypasses filters)
- Attractive proposal (laptop upgrade — social engineering)

**Fake login page identification:**

The attacker puts the legitimate brand name as a subdomain of their own
malicious domain:
```
Fake:  microsoftonline.login444123.com/signin
Real:  login.microsoftonline.com
```

Rule: the registered domain is the last two labels of the hostname — that
is what the attacker actually owns. Everything to the left of it is a
subdomain they control and can name anything.

```
https://microsoftonline.login444123.com/signin
        ^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^^
        subdomain        registered domain
        (attacker named  (attacker owns this)
        this anything)
```

---

## Challenges

Most content here was new — Security+ covers phishing definitions but not
analysis methodology. Email headers, SPF/DKIM/DMARC mechanics, punycode
detection, and the modern legitimate-platform attack flow were all first
exposure. The volume took two hours to absorb properly. Getting stuck on
which technique to classify as primary during triage was the main friction —
some emails combined three techniques at once and the overlap was ambiguous.

---

## Security+ Alignment

**Domain 1.0 - General Security Concepts (12%):** Security awareness,
phishing prevention, social engineering defense.

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Phishing,
social engineering, email-based threats, impersonation, typosquatting,
spoofing, malicious attachments.

**Domain 4.0 - Security Operations (28%):** Email security monitoring,
incident detection, threat intelligence, security controls (SPF/DKIM/DMARC).

---

## Evidence

![Email Header Analysis - Spoofing Detection](../07-Screenshots/Day12-1.png)
*Email headers showing SPF/DKIM/DMARC failures and Return-Path mismatch,
confirming spoofed sender address.*

![Phishing Email Triage Dashboard](../07-Screenshots/Day12-2.png)
*Triage dashboard showing classification of phishing emails with three
identified indicators per message.*

---

## Key Takeaways

**Phishing analysis workflow:**
```
1. Check sender domain — matches company domain? Free email service?
2. Read headers — Authentication-Results (SPF/DKIM/DMARC), Return-Path
3. Scan body — urgency language, authority claims, side channel redirect
4. Inspect links — hover to check real destination, verify domain
5. Check attachments — file extension, unexpected or disguised type
6. Identify 3 indicators — confirm classification
```

**Email authentication quick reference:**

| Result | Meaning |
|---|---|
| SPF pass | Sending server authorized for this domain |
| SPF fail | Sending server NOT authorized — spoofing likely |
| DKIM pass | Email signature valid, not tampered |
| DKIM fail | Signature invalid or missing |
| DMARC fail | Failed combined SPF + DKIM policy check |
| Return-Path ≠ From | `From:` address was forged |

**Punycode detection:**
```
Look for xn-- prefix in Return-Path header
xn--tbfc-xqa.com = punycode-encoded domain
tbfc.com = legitimate domain
```

**Fake login page domain rule:**
```
https://microsoftonline.login444123.com/signin

Registered domain = login444123.com   (attacker owns this)
microsoftonline   = subdomain         (attacker named it anything they want)

Check the last two labels of the hostname before the /
That is what someone actually registered and controls
```

**Malicious file type quick reference:**
```
.hta              → Outside browser sandbox, Windows Script Host
.docm / .xlsm     → Office macros
.js / .vbs        → Script files via Windows Script Host
.zip / .rar       → Archive hiding any of the above
```

**Key terms:**

| Term | Definition |
|---|---|
| Phishing | Targeted deception to steal credentials, deliver malware, or commit fraud |
| Spam | Bulk untargeted email; low risk, not a precision attack |
| SPF | DNS record defining authorized sending servers for a domain |
| DKIM | Cryptographic email signature proving message integrity and sender |
| DMARC | Policy using SPF + DKIM to accept, quarantine, or reject suspicious email |
| Return-Path | Actual envelope sender address — differs from From if spoofed |
| Typosquatting | Registering misspelled version of a legitimate domain |
| Punycode | ASCII encoding of Unicode characters; ACE prefix `xn--` |
| HTA | HTML Application — runs outside browser sandbox with system access |
| Side channel | Moving communication off monitored platforms to evade detection |
