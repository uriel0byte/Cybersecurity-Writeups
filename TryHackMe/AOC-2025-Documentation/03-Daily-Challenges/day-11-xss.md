# Day 11: Cross-Site Scripting (XSS) — Merry XSSMas

**Date:** December 11, 2025  
**Time Spent:** 1 hour  
**Difficulty:** ★★☆☆  
**Category:** Web Application Security / OWASP Top 10  
**Room:** https://tryhackme.com/room/xss-aoc2025-c5j8b1m4t6

---

## Overview

McSkidy's secure message portal was showing suspicious search terms and
script-like messages in the logs. The task was to investigate by exploiting
the portal's XSS vulnerabilities — first a Reflected XSS via the search
function, then a Stored XSS via the message form. XSS definition was already
known from Security+ theory; this was the first time seeing both variants
work in practice and writing actual payloads.

---

## What I Learned

### What is XSS?

XSS (Cross-Site Scripting) lets an attacker inject malicious JavaScript into
a web page that other users load in their browsers. It works because the
application fails to sanitize or encode user input before rendering it as HTML.

```
Attacker injects JS → App stores or reflects it → Victim's browser executes it
```

What the attacker can do once code runs in the victim's browser:
- Steal session cookies
- Hijack the session
- Perform actions as the victim
- Redirect to malicious sites
- Harvest credentials via fake login popups

### Reflected XSS (Non-Persistent)

The payload is injected into a request and immediately reflected back in the
response. Never stored on the server. Each victim has to be individually
tricked into clicking a crafted URL.

**Normal URL:**
```
https://trygiftme.thm/search?term=gift
```

**Malicious URL:**
```
https://trygiftme.thm/search?term=<script>alert(atob("VEhNe0V2aWxfQnVubnl9"))</script>
```

Attack flow:
```
1. Attacker crafts malicious URL
2. Victim clicks link (via phishing)
3. Server reflects payload back in response
4. Browser executes the JavaScript
```

`atob()` is a built-in JavaScript function that decodes Base64 strings. Used
here to obscure the payload from basic filters.

### Stored XSS (Persistent)

The payload is saved to the server — database, file system, message queue —
and executes automatically every time any user loads the affected page. No
social engineering required per victim after the initial injection.

**Malicious comment submission:**
```http
POST /post/comment HTTP/1.1
Host: tgm.review-your-gifts.thm

postId=3
name=Tony Baritone
email=tony@baritone.com
comment=<script>alert(atob("VEhNe0V2aWxfU3RvcmVkX0VnZ30="))</script>
```

Attack flow:
```
1. Attacker submits payload via form
2. Server stores it in the database
3. Every user who loads the page executes the script
4. Attacker's code runs without further action
```

### Reflected vs. Stored

| | Reflected | Stored |
|---|---|---|
| Persistence | Non-persistent | Stored on server |
| Delivery | Crafted URL (phishing) | Form submission |
| Victims | Targeted individuals | All users who view the page |
| Requires social engineering | Yes, per victim | No, once injected |
| Evidence on server | None | Payload in database/logs |
| Danger level | Lower | Higher |

### Preventing XSS

**1. Use `textContent` instead of `innerHTML`**

```javascript
element.innerHTML = userInput;    // DANGEROUS — executes HTML and JS
element.textContent = userInput;  // SAFE — renders as plain text only
```

**2. Secure cookie attributes**

```
HttpOnly   — Blocks JavaScript from reading the cookie entirely
Secure     — Cookie only sent over HTTPS
SameSite   — Blocks cookie from being sent in cross-site requests
```

Reduces damage from XSS by making session cookie theft much harder even if
a payload executes.

**3. Sanitize and encode output**

Encode special characters before rendering user input as HTML:
```
<  →  &lt;
>  →  &gt;
"  →  &quot;
'  →  &#x27;
```

Strip or escape elements that should never appear in user content:
- `<script>` tags
- Event handlers: `onclick`, `onerror`, `onload`
- JavaScript URLs: `javascript:`

---

## Challenges

No real difficulty. The room was straightforward with clear instructions and
immediate visual feedback from alert boxes. The main value was seeing both
XSS types work in practice after only knowing the definition from Security+
theory — understanding why Stored is more dangerous than Reflected only
clicked once the attack flow was visible rather than described.

---

## Security+ Alignment

**Domain 1.0 - General Security Concepts (12%):** Secure coding practices,
input validation, vulnerability types.

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Injection
attacks, XSS, OWASP Top 10, web application vulnerabilities.

**Domain 4.0 - Security Operations (28%):** Vulnerability assessment, security
monitoring, web application security, incident detection.

---

## Evidence

![XSS System Logs](../07-Screenshots/Day11.png)
*System logs showing XSS payload execution. Reflected XSS confirmed via alert
box on search function. Stored XSS payload persisted in message form and
executed on every subsequent page load.*

---

## Key Takeaways

**XSS payload quick reference:**
```javascript
// Basic proof-of-concept
<script>alert('XSS')</script>

// With Base64 encoding (bypasses basic filters)
<script>alert(atob("BASE64_STRING_HERE"))</script>

// Event handler vector (no script tags needed)
<img src=x onerror=alert('XSS')>

// Input field vector
"><script>alert('XSS')</script>
```

**Detection — what to look for in logs:**
```
<script>          in user input fields or URL parameters
alert(            in HTTP request logs
atob(             Base64-encoded payload indicator
onerror=          Event handler injection attempt
javascript:       JavaScript URL scheme in href values
eval(             Dynamic code execution attempt
```

**Reflected vs. Stored at a glance:**
```
Reflected: payload in URL → reflected in response → one victim at a time
Stored:    payload in form → saved to DB → every visitor is a victim
```

**Defensive checklist:**
```
textContent instead of innerHTML       (client-side rendering)
HttpOnly + Secure + SameSite cookies   (cookie protection)
Output encoding: < > " ' &            (HTML encoding)
Strip <script>, onclick, javascript:   (input sanitization)
Content Security Policy (CSP) header   (browser-level enforcement)
WAF rules for XSS patterns             (network-level detection)
```

**Key terms:**

| Term | Definition |
|---|---|
| XSS | Injection of malicious JavaScript into pages viewed by other users |
| Reflected XSS | Payload reflected immediately in response, not stored |
| Stored XSS | Payload saved to server, executes for every user who loads the page |
| `innerHTML` | DOM property that parses and renders HTML — dangerous with user input |
| `textContent` | DOM property that renders as plain text — safe alternative |
| `HttpOnly` | Cookie attribute blocking JavaScript access |
| `atob()` | JavaScript function decoding Base64 strings |
| CSP | Content Security Policy — browser-enforced header restricting script sources |
| WAF | Web Application Firewall — filters malicious HTTP requests |
| OWASP Top 10 | Industry standard list of critical web application security risks |
