# Day 2: Merry Clickmas - Phishing

**Date:** December 2, 2025  
**Time Spent:** 2 hours  
**Difficulty:** ★★☆☆  
**Category:** Social Engineering / Red Team  
**Room:** https://tryhackme.com/room/phishing-aoc2025-h2tkye9fzU

---

## Overview

Authorized red team exercise alongside Recon McRed, Exploit McRed, and Pivot McRed.
The objective was to test whether TBFC employees would fall for a phishing email and
enter credentials on a fake login page, validating whether security awareness training
was actually working.

---

## What I Learned

### Social Engineering Fundamentals

Social engineering targets people, not systems. Technical defenses fail when users are
manipulated into bypassing them.

**Psychological triggers attackers use:**
- Urgency: "Act now or lose access!"
- Curiosity: "You've received a package!"
- Authority: "Your manager needs this immediately!"

### Phishing Types

- **Phishing** - Email based, still the most common
- **Smishing** - SMS/text message
- **Vishing** - Voice call
- **Quishing** - QR code (emerging)

### S.T.O.P. Mnemonics

**Before clicking:**
```
S - Suspicious? Does something feel off?
T - Telling me to click? Is there pressure?
O - Offering something? Too good to be true?
P - Pushing urgency? Must act right now?
```

**Response protocol:**
```
S - Slow down (scammers rely on panic)
T - Type addresses yourself, never use the link
O - Open nothing unexpected without verifying
P - Prove the sender, check real address not display name
```

These mnemonics were part of TBFC's security awareness training. Employees still fell
for the phishing test. Training alone is not enough without regular testing.

### Building the Phishing Trap

**Step 1 - Deploy the credential harvesting server:**

```bash
cd ~/Rooms/AoC2025/Day02
./server.py
# Output: Starting server on http://0.0.0.0:8000
```

Hosts a fake TBFC login page. Any credentials entered get captured server-side.
Test locally at `http://127.0.0.1:8000` before sending the campaign.

**Step 2 - Craft the pretext:**

Impersonating Flying Deer, a shipping company TBFC employees already trust and
regularly communicate with. Subject: "Shipping Schedule Changes." Professional tone,
plausible business reason, no attachments, no obvious red flags.

Why it works:
- Employees expect communication from Flying Deer
- Shipping schedules actually change during busy seasons
- Urgency is implied without being loud about it

### Social-Engineer Toolkit (SET)

```bash
setoolkit
```

**Menu navigation:**
```
1 - Social-Engineering Attacks
5 - Mass Mailer Attack
1 - E-Mail Attack Single Email Address
```

**Configuration:**
```
Send email to:   factory@wareville.thm
Delivery method: Use your own server or open relay (option 2)
From address:    updates@flyingdeer.thm
From name:       Flying Deer
SMTP server:     MACHINE_IP
Port:            25
High priority:   no
Attach file:     n
Subject:         Shipping Schedule Changes
```

**Email body:**
```
Dear elves,
Kindly note that there have been significant changes to the shipping
schedules due to increased shipping orders.
Please confirm the new schedule by visiting http://CONNECTION_IP:8000
Best regards,
Flying Deer
```

Type `END` on a new line to finish. One mistake means `Ctrl+C` and starting over.
Option 99 returns to the main menu.

Wait 1-2 minutes after sending for the simulated user to interact.

### Results

Admin credentials captured from the fake login page. Tested reuse on the real TBFC
email portal at `http://MACHINE_IP`. Password was reused. Accessed admin's mailbox
and found sensitive information inside.

**What this demonstrates:**
- Phishing bypassed security awareness training
- Password reuse enabled access beyond the initial target
- Admin access from a single phishing email is a realistic outcome

### Email Spoofing and SMTP

SMTP does not verify sender identity by default. Without SPF, DKIM, and DMARC,
anyone can send email claiming to be anyone. The display name is what users see.
Most never check the actual sending address.

**Email authentication controls:**
- **SPF** - Defines which servers are allowed to send for a domain
- **DKIM** - Cryptographic signature that verifies the message
- **DMARC** - Policy for what to do when SPF/DKIM fail

---

## Challenges

SET has multiple nested menus with no way to go back mid-configuration. One wrong
answer and you restart from the top with `Ctrl+C`. Patience and reading each prompt
carefully is the only way through it.

Crafting the email required thinking through what employees would actually expect to
receive. Obvious urgency raises flags. The goal was a request that felt routine.

---

## Security+ Alignment

**Domain 1.0 - General Security Concepts (12%):** Social engineering tactics, security
awareness training, authentication security.

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Phishing attack
techniques, credential harvesting, spear-phishing, email-based attacks.

**Domain 4.0 - Security Operations (28%):** Email security, incident detection,
credential compromise investigation, user behavior monitoring.

---

## Key Takeaways

**Phishing detection indicators:**
- Display name does not match actual sending address
- Sender domain is misspelled (microsft.com)
- Reply-to address differs from the from address
- SPF/DKIM/DMARC checks failing
- Generic greeting instead of your actual name
- Suspicious link destination on hover
- Unexpected attachments
- Urgent threats or too-good-to-be-true offers

**SET quick reference:**
```bash
setoolkit
# 1 > 5 > 1 > configure > END to finish body
# Ctrl+C to restart, 99 for main menu
```

**Credential harvesting server:**
```bash
cd ~/Rooms/AoC2025/Day02
./server.py                         # Starts on port 8000
firefox http://127.0.0.1:8000      # Test before sending
```

**Credential compromise response checklist:**
1. Lock compromised account immediately
2. Force password reset
3. Review login history for unauthorized access
4. Check for password reuse on other systems
5. Assess what data was accessed
6. Enforce MFA if not already in place
7. Alert the security team

**Authorized testing rules:**
- Written authorization required before any test
- Defined scope only, no out-of-scope targets
- No destructive payloads
- Have an incident response plan ready if a user reports the test
