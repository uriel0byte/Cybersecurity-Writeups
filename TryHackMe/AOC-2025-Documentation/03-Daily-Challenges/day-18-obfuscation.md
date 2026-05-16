# Day 18: Obfuscation — The Egg Shell File

**Date:** December 18, 2025  
**Time Spent:** 1 hour  
**Difficulty:** ★★★☆ *(Official rating: Medium — covered quickly; theory familiar from Security+ and Day 17, practical was new)*  
**Category:** Obfuscation / Malware Analysis / PowerShell  
**Room:** https://tryhackme.com/room/obfuscation-aoc2025-e5r8t2y6u9

---

## Overview

McSkidy intercepts a phishing email spoofing `northpole-hr` (which doesn't exist —
TBFC HR is at the South Pole). The attached PowerShell script, `SantaStealer.ps1`,
is full of unreadable strings. The task: deobfuscate a C2 URL hidden inside it,
then flip roles and obfuscate an API key using XOR to understand how attackers
apply the same technique.

Two-part room. First half is defensive (peel the obfuscation off). Second half is
offensive (put it back on). Short room — the theory from Day 17 carried over directly.

---

## What I Learned

### Encoding vs. Encryption vs. Obfuscation

Building on Day 17's encoding/encryption distinction with obfuscation added:

| Aspect | Encoding | Encryption | Obfuscation |
|---|---|---|---|
| Purpose | Data compatibility | Confidentiality | Hide meaning, evade detection |
| Key required | No | Yes | No |
| Security | None | Yes | Weak — delays analysis, doesn't prevent it |
| Recovery method | Decode (trivial) | Need the key | Manual analysis or tooling |
| Examples | Base64, Hex | AES, RSA | ROT ciphers, XOR, layering |

**The critical point about obfuscation:** it is not a security measure. It is a
deterrent. A determined analyst will get through it — the goal is to slow automated
detection tools and make analysis expensive, not to make reversal impossible.

Attackers use it specifically to bypass rules like:
```
if (file contains "http://c2server.com") → block
```

So the C2 URL gets ROT13'd, XOR'd, or Base64'd at rest and reconstructed at runtime.
The script still works. The signature scanner sees nothing it recognizes.

### The Four Techniques This Room Covers

**ROT1**

Shifts each letter one position forward in the alphabet. A→B, B→C, Z→A. Spaces
stay unchanged.

```
Original:   carrot coins go brr
ROT1:       dbsspu dpjot hp css
```

Visual tell: common words look exactly "one letter off". Easy to spot once you
know what to look for.

---

**ROT13**

Same concept but shifts 13 positions. Self-reversing — applying ROT13 twice
returns the original. (Covered in Day 17 as well.)

Visual tell: check three-letter common words.
```
the → gur
and → naq
```
Spaces stay in place. If you see those patterns, it's ROT13.

---

**Base64**

Encodes binary data as printable ASCII. Covered in detail in Days 15 and 17.

Visual tell: long alphanumeric string containing `+` and `/`, often ending in
`=` or `==`. The `=` padding at the end is the most reliable quick indicator.

---

**XOR**

Each character is converted to a byte, then combined with a key byte using the
XOR operation. Covered in Day 17 — the self-reversing property applies here too:
XOR the output with the same key and you get the original back.

Visual tell: looks like random symbols, but the output is **exactly the same
length** as the input. If you notice a short repeating pattern every few
characters, the key is shorter than the data (key reuse).

```
CyberChef XOR settings:
- Key: paste the key value
- Key type: set to HEX if the key is hex, UTF-8 if it's a string
```

---

**Layered obfuscation — the important pattern**

Real malware chains these. The room shows this example:

```
Original → gzip → XOR with key 'x' → Base64 → stored/transmitted
```

To reverse, apply the **inverse operations in reverse order**:

```
Base64 encoded string → [From Base64] → [XOR with 'x'] → [Gunzip] → Original
```

This is the core mental model for de-obfuscation: figure out what was done, then
undo it in the opposite order. CyberChef makes this mechanical once you know the
chain.

---

### The Practical: SantaStealer.ps1

**Script location:** `C:\Users\Administrator\Desktop\SantaStealer.ps1`  
**Open with:** Visual Studio Code (double-click from desktop, takes a moment to load)  
**Navigate to:** `# ===== Start here =====` section — comments guide you through both parts

---

**Part 1 — Deobfuscate the C2 URL**

The script contains an obfuscated C2 URL stored as a mangled string. The comments
tell you the obfuscation method. De-obfuscate it using CyberChef, replace the
obfuscated value in the script with the plaintext URL, save, then run.

```powershell
cd C:\Users\Administrator\Desktop\
.\SantaStealer.ps1
```

Script confirms the decoded URL is correct and outputs the flag.

**Flag 1:** `THM{C2_De0bfuscation_29838}`

This mirrors exactly what defenders do during incident response when they find
a PowerShell loader with an obfuscated beacon URL.

---

**Part 2 — Obfuscate the API Key with XOR**

Now the roles flip. The script has a plaintext API key and the comments tell you
to obfuscate it using XOR with a specified key, then paste the result back into
the script. Run it again — the script de-XORs its own value at runtime and confirms
the key is correct.

```
CyberChef workflow:
1. Input: plaintext API key
2. Operation: XOR with the key from the comments
3. Set key type to match (UTF-8 or HEX as specified)
4. Copy output → paste into script → save → run
```

**Flag 2:** `THM{API_Obfusc4tion_ftw_0283}`

The value of this half: it forces you to think like the attacker for a minute.
Obfuscating a credential in a script to avoid string-match detection is a real
technique in actual malware. Understanding how it is applied makes it easier to
recognize and reverse when you encounter it in the field.

---

## Challenges

Short room with a light challenge load. The theory was already covered — Day 17
laid the encoding/XOR groundwork, Security+ covered the conceptual distinction.
What the room added was the two-sided exercise: deobfuscating something to get
a flag, then obfuscating something yourself. That second part was the more
interesting one because it forces a different mental model.

The main confusion that carried over from Day 17: when does XOR count as encryption
versus obfuscation? The answer is intent and key management. XOR with a properly
managed secret key is encryption. XOR hardcoded in a script where the key is visible
in the same file is obfuscation — it provides no real security, just a small barrier
to automated detection.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Obfuscation
techniques, malware evasion, indicator analysis.

**Domain 3.0 - Cryptography & PKI (12%):** Encoding vs. encryption vs. obfuscation,
XOR operations, cipher techniques.

---

## Evidence

![CyberChef C2 Deobfuscation](../07-Screenshots/Day18-1.png)
*CyberChef recipe reversing the C2 URL obfuscation — plaintext URL recovered and
confirmed by running SantaStealer.ps1.*

![XOR API Key Obfuscation](../07-Screenshots/Day18-2.png)
*XOR operation applied to API key in CyberChef — obfuscated value pasted back into
script and confirmed by running SantaStealer.ps1 again.*

---

## Key Takeaways

**Three-way distinction (updated from Day 17):**

| | Encoding | Encryption | Obfuscation |
|---|---|---|---|
| Key needed | No | Yes | No |
| Security | None | Yes | Weak |
| Recovery | Decode | Need key | Analysis |
| Attacker use | Payload delivery | C2 traffic | Detection evasion |

**Visual recognition cues:**

| Technique | What it looks like |
|---|---|
| ROT1 | Words look one letter off; spaces unchanged |
| ROT13 | `the` → `gur`, `and` → `naq`; spaces unchanged |
| Base64 | Alphanumeric + `+` `/`; ends in `=` or `==` |
| XOR | Random symbols; same length as input; may show short repeating pattern |

**De-obfuscation order rule:**  
Always reverse the operations in reverse order. If something was obfuscated as
`gzip → XOR → Base64`, the recipe is `From Base64 → XOR → Gunzip`.

**CyberChef XOR settings:**
- Set key type to **HEX** if the key is in hex format
- Set key type to **UTF-8** if the key is a plaintext string
- Getting this wrong produces garbage output

**Flags:**

| Task | Flag |
|---|---|
| Part 1 — Deobfuscate C2 URL | `THM{C2_De0bfuscation_29838}` |
| Part 2 — Obfuscate API key with XOR | `THM{API_Obfusc4tion_ftw_0283}` |

**Key terms:**

| Term | Definition |
|---|---|
| Obfuscation | Making code or data hard to read while keeping it functional — not a security control |
| ROT1 / ROT13 | Caesar-style substitution ciphers; ROT13 is self-reversing |
| XOR obfuscation | XOR with a hardcoded key in the same file — evades signature detection, not real encryption |
| C2 (Command & Control) | Attacker infrastructure the malware phones home to; often obfuscated to avoid domain blocking |
| SantaStealer.ps1 | The PowerShell script used in this room; contains an obfuscated C2 URL and an API key to obfuscate |
| Layered obfuscation | Chaining multiple techniques (e.g., gzip → XOR → Base64) to make reversal more laborious |
