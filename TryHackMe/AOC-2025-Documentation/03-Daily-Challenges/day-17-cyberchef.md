# Day 17: CyberChef — Hoperation Save McSkidy

**Date:** December 17, 2025  
**Time Spent:** 2 hours  
**Difficulty:** ★★★☆ *(Official rating: Easy/Medium — prior CTF crypto experience helped; web inspection via developer tools was new in practice)*  
**Category:** Encoding / Decoding / Cryptography / Web Inspection  
**Room:** https://tryhackme.com/room/encoding-decoding-aoc2025-s1a4z7x0c3

---

## Overview

McSkidy is trapped in King Malhare's Quantum Warren behind five security locks.
Each lock uses a different encoding or transformation scheme. Breaking them required
inspecting web app headers and JavaScript login logic through browser developer tools,
then building CyberChef recipes to reverse whatever encoding the login system applied
to the guard's password response.

Prior CTF experience with cryptography covered the theory. The new part was practical
web inspection — reading HTTP headers and JavaScript source in the Debugger tab to
figure out what to decode before touching CyberChef.

---

## What I Learned

### Encoding vs. Encryption

This distinction is fundamental and gets blurred constantly.

| Aspect | Encoding | Encryption |
|---|---|---|
| Purpose | Data compatibility | Confidentiality |
| Key required | No | Yes |
| Security | None — anyone can reverse it | Yes — only key holder can reverse |
| Examples | Base64, Hex, ROT13 | AES, RSA, XOR with secret key |
| Common mistake | Treating it as security | — |

**The core point:** Base64 is not security. It is a formatting choice. If something
is Base64 encoded, anyone can decode it in five seconds. Seeing encoded data in an
HTTP header or JavaScript file is not protection — it is just slightly inconvenient
to read.

### CyberChef Interface

Web-based tool for encoding, decoding, encryption, decryption, and data
transformation. Available online at https://gchq.github.io/CyberChef/ or offline
from the AttackBox bookmarks.

| Area | Purpose |
|---|---|
| Operations (left panel) | Library of 300+ transformation functions |
| Recipe (center) | Drag operations here to build a chain |
| Input (bottom-left) | Paste the data to transform |
| Output (right) | Result after the recipe runs |

**Key capability:** Operations chain left to right. Output of one feeds into the
next. This is what makes reversing multi-step encoding possible — you reconstruct
the reverse of whatever the login logic applied, in the same order.

Toggle individual operations on/off using the middle button on each recipe step.
Useful when testing which step in the chain is causing problems.

### Browser Developer Tools

For this room, two tabs matter:

**Network tab** — shows HTTP requests and responses including all headers.

```
Open dev tools → Network tab → Refresh the page
→ Click the first response in the list
→ Check Response Headers for custom fields (X-Magic, Recipe-ID, etc.)
```

Always refresh after opening the Network tab. Opening it after the page loads
means the initial request is already gone.

**Debugger tab** — shows JavaScript source files loaded by the page.

```
Open dev tools → Debugger tab
→ Find app.js or the relevant script file
→ Read the login validation function
→ Comments in the code tell you exactly what encoding to reverse
```

The login logic comments in this room were explicit: they named the operations and
order. Real-world code usually won't be this clear, but reading JS login logic to
understand what a system does with a password before checking it is a real skill.

---

## What I Learned

### The Five Locks

**General workflow for every lock:**
```
1. Find guard name on the page → Base64 encode it → this is the username
2. Check Network tab headers → find magic question or Recipe ID
3. Check Debugger tab → read login logic comments
4. Chat the guard (Base64-encoded message) → get encoded password back
5. Build CyberChef recipe to reverse the login logic
6. Log in with Base64 username + decoded plaintext password
```

**Chat note:** All chat messages must be Base64 encoded to communicate with guards.
Include a question mark in messages to get a reply. From Lock 3 onwards, guards
take 2–3 minutes to respond — keep messages short. `Password please.` works.

---

**Lock 1 — Outer Gate: Simple Base64**

Login logic: `Base64(password)`  
Magic question (from Network tab header): `What is the password for this level?`

```
Guard reply → [From Base64] → Plaintext password
```

This is the baseline. One decode operation, no tricks.  
Lock 1 password: `Iamsofluffy`

---

**Lock 2 — Outer Wall: Double Base64**

Login logic: `Base64(Base64(password))`  
Magic question (from Network tab header): `Did you change the password?`

The password is encoded twice before being stored. You have to decode twice.

```
Guard reply → [From Base64] → Still looks encoded
             → [From Base64] → Plaintext password
```

CyberChef recipe: `From Base64 → From Base64`  
Lock 2 password: `Itoldyoutochangeit!`

---

**Lock 3 — Guard House: XOR + Base64**

Login logic: `Base64(XOR(password, key))`  
No magic question from this lock onward — just ask the guard for the password.  
XOR key (from JS source): `cyberchef`

**What XOR is:** A bitwise operation applied between data and a key. The critical
property for forensics: XOR is its own inverse. XOR the result with the same key
again and you get the original data back.

```
plaintext XOR key = ciphertext
ciphertext XOR key = plaintext   ← same key, same operation, reversed
```

XOR alone is not encryption. Without keeping the key secret, it provides no
security. But it appears everywhere in obfuscation and in combination with other
operations, so understanding reversibility matters.

To reverse Lock 3:
```
Guard reply → [From Base64] → XOR result
            → [XOR with key: cyberchef] → Plaintext password
```

CyberChef recipe: `From Base64 → XOR (key: cyberchef)`  
Lock 3 password: `BugsBunny`

---

**Lock 4 — Inner Castle: MD5 Hash + Base64**

Login logic: `Base64(MD5(password))`  

**What MD5 is:** A cryptographic hash function. Takes any input, produces a fixed
32-character hexadecimal string. One-way — you cannot mathematically reverse a hash
to recover the original input.

```
"Password123" → MD5 → "482c811da5d5b4bc6d497ffa98491e38"
```

Same input always produces the same hash (deterministic). Different inputs produce
completely different hashes. This is why MD5 is used to verify file integrity and
was historically used to store passwords.

**Why MD5 is broken for passwords:** Attackers precompute hashes for millions of
common passwords and store them in lookup tables (rainbow tables). You look up the
hash, get the original password back. This is not cracking — it is a database lookup.

**To reverse Lock 4:**
```
Guard reply → [From Base64] → MD5 hash string
            → Paste hash into CrackStation (crackstation.net)
            → CrackStation returns plaintext if it's in the database
```

CyberChef recipe: `From Base64` (to extract the hash), then CrackStation for lookup.  
Lock 4 password: `passw0rd1`

This is why modern password storage uses salted hashing algorithms like bcrypt or
Argon2 — salting defeats precomputed tables entirely.

---

**Lock 5 — Prison Tower: Variable Recipes**

Login logic: varies by Recipe ID found in the page headers.

Check the Network tab for the `Recipe-ID` header value, then match to the
corresponding decode chain below.

| Recipe ID | CyberChef Chain |
|---|---|
| 1 | From Base64 → Reverse → ROT13 |
| 2 | From Base64 → From Hex → Reverse |
| 3 | ROT13 → From Base64 → XOR (extracted key, **UTF-8 mode**) |
| 4 | ROT13 → From Base64 → ROT47 |

**ROT13:** Shifts each letter 13 positions in the alphabet. A→N, B→O, Z→M.
Applying ROT13 twice returns the original text. No key needed — it is reversible
with the same operation. Zero security value; used purely for obfuscation.

**ROT47:** Same concept as ROT13 but covers all printable ASCII characters
(letters, numbers, symbols). Also self-reversing.

**XOR UTF-8 note:** When using XOR in CyberChef for Recipe ID 3, set the XOR
operation to **UTF-8 mode** or the output will be wrong.

**Lock 5 workflow:**
```
1. Note encoded guard name from page
2. Check Network tab → find Recipe ID header value
3. Build corresponding recipe in CyberChef
4. Send "Password please." to guard (Base64 encoded)
5. Paste guard reply into CyberChef recipe
6. Log in with result
```

Lock 5 password: `51rBr34chBl0ck3r`  
Final flag: `THM{M3D13V4L_D3C0D3R_4D3P7}`

---

## Challenges

The cryptography theory — Base64, XOR, MD5, ROT ciphers — was already familiar
from picoCTF and other CTF work. The gap was practical web inspection. Finding
the right header in the Network tab response list, reading JavaScript in the
Debugger tab, and connecting what the code said to what CyberChef needed to do
was new in an applied context.

Lock 5 was the most demanding because the decode chain changes based on the Recipe
ID. Getting the order wrong produces garbage output, and the XOR UTF-8 mode
requirement is not obvious unless you know to look for it.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Obfuscation
techniques, cryptographic concepts, encoding vs. encryption.

**Domain 3.0 - Cryptography & PKI (12%):** Hashing algorithms, symmetric
operations, encoding formats, cryptographic use cases.

**Domain 4.0 - Security Operations (28%):** Data analysis tools, evidence
decoding, web application inspection, incident response techniques.

---

## Key Takeaways

**Encoding vs. Encryption (never mix these up):**

| | Encoding | Encryption |
|---|---|---|
| Key required | No | Yes |
| Security | None | Yes |
| Reversible by anyone | Yes | No |
| Examples | Base64, Hex, ROT13 | AES, XOR with secret key |

**CyberChef common operations:**

| Operation | What It Does |
|---|---|
| `From Base64` | Decode Base64 string |
| `To Base64` | Encode to Base64 |
| `From Hex` | Convert hex to ASCII |
| `XOR` | XOR with a key (set UTF-8 mode when needed) |
| `ROT13` | Rotate letters 13 positions (self-reversing) |
| `ROT47` | Rotate printable ASCII 47 positions (self-reversing) |
| `Reverse` | Reverse string order |
| `MD5` | Compute MD5 hash |

**Lock decode cheat sheet:**

| Lock | Encoding Scheme | CyberChef Recipe | Password |
|---|---|---|---|
| 1 | Base64 | `From Base64` | `Iamsofluffy` |
| 2 | Base64 × 2 | `From Base64 → From Base64` | `Itoldyoutochangeit!` |
| 3 | XOR(key: cyberchef) → Base64 | `From Base64 → XOR (cyberchef)` | `BugsBunny` |
| 4 | MD5 → Base64 | `From Base64` → CrackStation | `passw0rd1` |
| 5-R1 | Base64 → Reverse → ROT13 | `From Base64 → Reverse → ROT13` | — |
| 5-R2 | Base64 → Hex → Reverse | `From Base64 → From Hex → Reverse` | — |
| 5-R3 | ROT13 → Base64 → XOR | `ROT13 → From Base64 → XOR (UTF-8)` | — |
| 5-R4 | ROT13 → Base64 → ROT47 | `ROT13 → From Base64 → ROT47` | — |

Lock 5 password (varies by Recipe ID): `51rBr34chBl0ck3r`  
Final flag: `THM{M3D13V4L_D3C0D3R_4D3P7}`

**Developer tools quick reference:**

| Tab | What to look for |
|---|---|
| Network | Custom response headers (X-Magic, Recipe-ID) — refresh page first |
| Debugger | Login validation logic in app.js — read comments for encoding steps |
| Inspector | Hidden input fields, element attributes |

**Hash cracking tools:**

| Tool | Type | Use |
|---|---|---|
| CrackStation | Online | Common passwords — fast first check |
| HashCat | Offline | Large-scale, GPU-accelerated |
| John the Ripper | Offline | Dictionary and rule-based attacks |

**Key terms:**

| Term | Definition |
|---|---|
| Encoding | Data transformation for compatibility — no key, no security, anyone can reverse |
| Encryption | Data transformation for confidentiality — requires a key to reverse |
| Base64 | Encoding scheme representing binary data as printable ASCII characters |
| XOR | Bitwise operation; self-reversing with the same key — not encryption on its own |
| MD5 | One-way hash function producing a 32-char hex string; broken for passwords due to rainbow tables |
| Rainbow table | Precomputed database mapping hash values back to their plaintext inputs |
| ROT13 | Caesar cipher shifting letters 13 positions; applying it twice returns the original |
| ROT47 | ROT13 variant covering all printable ASCII characters |
| Recipe | CyberChef term for a saved chain of chained operations |
| Debugger tab | Browser dev tools panel showing JavaScript source files and application logic |
