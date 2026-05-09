# Day 5: Santa's Little IDOR

**Date:** December 5, 2025  
**Time Spent:** 3 hours  
**Difficulty:** ★★★☆  
**Category:** Web Security / Access Control Vulnerabilities  
**Room:** https://tryhackme.com/room/idor-aoc2025-zl6MywQid9

---

## Overview

A suspiciously named account called "Sir Carrotbane" had accumulated unauthorized
vouchers on the TryPresentMe website. Parents could not activate legitimate vouchers
and were receiving phishing emails containing their private information. The
investigation traced back to IDOR vulnerabilities across four different reference
types — numeric IDs, Base64 encoding, MD5 hashes, and UUID v1 timestamp exploitation.

---

## What I Learned

### Authentication vs. Authorization

These get conflated constantly and the difference matters.

**Authentication:** Verifying who you are. Supplying credentials returns a session
token. That token is sent with every subsequent request, not just login.

**Authorization:** Verifying what you are allowed to do. Cannot happen before
authentication. Must be enforced server-side on every request, not just once.

```
Authentication  = Who are you?
Authorization   = What can you access?
```

Session expiry redirects to login for reauthentication. The token is the mechanism
that carries authentication state across requests.

### What IDOR Is

IDOR (Insecure Direct Object Reference) is an access control vulnerability where an
application uses IDs to retrieve data without checking if the requesting user is
actually authorized to access it.

```
URL: https://site.com/TrackPackage?packageID=1001
Change to: https://site.com/TrackPackage?packageID=1002
Result: You see someone else's package details
```

The vulnerable SQL pattern behind most IDOR:
```sql
SELECT person, address, status FROM Packages WHERE packageID = value;
-- No check: "Does this user own packageID 1002?"
```

The room notes that "Authorization Bypass" is technically more accurate. IDOR is the
industry-standard name but the root cause is always missing authorization checks.

### Four IDOR Types

**1. Numeric ID**
- Found `user_id=10` in Browser DevTools under Local Storage
- Changed to `user_id=11` and immediately accessed another user's account
- Sequential IDs are predictable by design

**2. Base64 Encoded ID**
- Network request showed `Mg==` for "view child"
- Decoded: `Mg== = 2`
- Encoding is not security. It is obfuscation. The underlying ID is still there.

**3. MD5 Hashed ID**
- References appeared as MD5 hashes
- If the original values are sequential or predictable, you can replicate the hash
- Hash identifier tools reveal the algorithm used

**4. UUID v1**
- Voucher codes used UUID v1 format
- UUID v1 embeds timestamp information in the ID itself
- Vouchers generated between 20:00-24:00 produce 14,400 possible UUIDs
- Brute force the entire range to find the valid one

### Privilege Escalation Types

**Horizontal:** Access another user's data at the same permission level.
Example: View your account → view someone else's account.

**Vertical:** Gain access to higher-privilege functionality.
Example: Normal user performing admin actions.

Most IDOR cases are horizontal.

### Fixing IDOR

**What actually works:**
- Server-side authorization check on every request: "Does this user own this item?"
- Unpredictable IDs (UUID v4, not v1) for public-facing references
- Log and alert on failed authorization attempts

**What does not work:**
- Base64 encoding
- MD5 hashing
- Making IDs appear random
- Any client-side obfuscation

If the server does not verify authorization, the vulnerability exists regardless of
how the ID looks.

---

## Challenges

The authentication versus authorization distinction was not immediately obvious,
specifically that authentication happens on every request through the session token
rather than just at login. That took a while to properly settle.

Web security is not my preferred domain. This room took 3 hours and burned me out a
bit. I am not confident in web topics yet but the foundation is necessary to build.

UUID v1 timestamp exploitation was a new concept entirely. The logic of generating
14,400 UUIDs across a 4-hour window made sense, but it took time to process.

---

## Security+ Alignment

**Domain 1.0 - General Security Concepts (12%):** Authentication vs. authorization,
access control models, least privilege.

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** OWASP Top 10,
web application vulnerabilities, access control weaknesses, privilege escalation.

**Domain 3.0 - Security Architecture (18%):** Web application security architecture,
session management, authorization enforcement.

---

## Key Takeaways

**IDOR in one line:**
```
Application uses IDs to retrieve data + no server-side ownership check = anyone
can access anyone's data
```

**Authentication vs. Authorization:**

| Concept | Question | Enforced |
|---|---|---|
| Authentication | Who are you? | Every request via session token |
| Authorization | What can you access? | Every request, server-side |

**IDOR types quick reference:**

| Type | Example | Exploit method |
|---|---|---|
| Numeric | `user_id=10` | Increment or decrement |
| Base64 | `Mg==` | Decode, change value, re-encode |
| MD5 hash | `cfcd208...` | Identify algorithm, hash new values |
| UUID v1 | `abc-def-...` | Extract timestamp, generate full range |

**Detecting IDOR in logs (SOC):**
```
# Sequential access pattern
GET /account/10  200 OK
GET /account/11  200 OK
GET /account/12  200 OK
← IDOR enumeration in progress

# UUID brute force pattern
POST /vouchers/claim  400 Bad Request x14399
POST /vouchers/claim  200 OK x1
← UUID v1 timestamp brute force
```

**WAF detection rules:**
- Alert on more than 5 sequential IDs from the same IP in a short window
- Flag rapid requests to the same endpoint with incrementing parameter values
- Monitor clusters of 400 errors followed by a 200 on voucher or account endpoints

**SDOR checklist for reporting findings:**
- Server-side ownership check on every request
- Use UUID v4 for public-facing references, never v1
- Never rely on encoding or hashing for access control
- Log failed authorization attempts and alert on patterns
- Test by attempting to access another authenticated user's resources

**OWASP context:**  
Broken Access Control has been the number one vulnerability in the OWASP Top 10
since 2021. It includes IDOR, missing function-level access control, privilege
escalation, CORS misconfiguration, and JWT manipulation. It sits at number one
because it is easy to exploit, hard to detect without logging, and causes direct
data exposure.
