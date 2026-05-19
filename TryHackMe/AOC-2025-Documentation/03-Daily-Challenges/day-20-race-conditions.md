# Day 20: Race Conditions — Toy to The World

**Date:** December 20, 2025  
**Time Spent:** 1 hour  
**Difficulty:** ★★★☆ *(Official rating: Medium — quick room; race condition theory was familiar from Security+, the new part was the specific Burp Suite parallel request workflow)*  
**Category:** Web Application Security / Race Conditions / Burp Suite  
**Room:** https://tryhackme.com/room/race-conditions-aoc2025-d7f0g3h6j9

---

## Overview

TBFC launched a limited-edition SleighToy with exactly 10 units available at
midnight. Within seconds, more than 10 customers received order confirmations.
Sir Carrotbane's Bandit Bunnies flooded the checkout with parallel requests, slipping
through a timing flaw before the stock counter could update. Result: oversold
inventory, negative stock values, and a broken store.

The task was to reproduce the exploit — capture a legitimate checkout request in
Burp Suite, duplicate it 15 times, and fire all copies simultaneously to trigger
the race condition.

---

## What I Learned

### What is a Race Condition

A race condition occurs when two or more processes access and modify a shared
resource simultaneously, and the final outcome depends on which one completes first.
The developer wrote the logic assuming one operation at a time. In reality, a burst
of simultaneous requests hits the server before any of them can see each other's
changes.

In this room: the stock check and stock deduction are two separate operations. If
15 requests all pass the stock check before any of them has deducted, the server
approves 15 orders against a stock of 10.

**Three types of race conditions covered:**

**TOCTOU (Time of Check to Time of Use):** The system checks a condition (stock > 0)
and then acts on it (deduct stock) as two separate steps. In the gap between check
and action, another request slips through the same check before the first deduction
has committed. Both proceed.

**Shared resource:** Multiple threads or requests write to the same resource
(a stock counter, a balance) without locking it. The final value reflects only the
last write, not all the intended deductions.

**Atomicity violation:** Operations that should execute as a single indivisible unit
are split across multiple steps. Any request that lands between those steps sees
an inconsistent intermediate state.

All three describe the same root cause from different angles: non-atomic operations
on shared data under concurrent load.

### Burp Suite — Parallel Request Attack

**Setup:**

```
1. Open Firefox → FoxyProxy icon → select Burp profile
2. Launch Burp Suite → Temporary project → Start Burp
3. Proxy tab → Intercept sub-tab → click until it reads "Intercept off"
   (if left on, every request hangs waiting for manual forwarding)
```

**Capture the target request:**

```
1. Visit http://MACHINE_IP
2. Login: attacker / attacker@123
3. Dashboard shows SleighToy Limited Edition — 10 units available
4. Add to Cart → Checkout → Confirm & Pay
5. Order succeeds — this creates a POST /process_checkout in HTTP history
6. Proxy tab → HTTP History → find POST /process_checkout
7. Right-click → Send to Repeater
```

**Build the parallel attack group:**

```
1. In Repeater, right-click the request tab
2. Add tab to group → Create tab group → name it "cart" → Create
3. Right-click the tab again → Duplicate tab → enter 15
   (15 identical requests in the cart group)
4. Send dropdown → "Send group in parallel (last-byte sync)"
```

**What "last-byte sync" means:** Burp prepares all 15 requests simultaneously and
holds them until the last byte of each is ready to send. It then fires all 15 at
the exact same moment. This maximizes timing overlap — all requests hit the server
before any stock deduction has committed to the database.

**Result:** Multiple order confirmations for the same item. Stock goes negative
(10 original units, 15 orders processed). Both flags appear on the orders page.

### Why Standard Browsers Can't Do This

Browsers send requests sequentially — one completes before the next begins. The
race window is measured in milliseconds. Sequential requests always land after the
previous deduction has committed. Burp's parallel send is the only way to land
multiple requests within the same timing window reliably.

### Blue Team Perspective

From a defender's view, a race condition attack looks like:

- Multiple identical POST requests from the same session in a very short time window
- Stock values going negative in the database
- Order count exceeding available inventory
- No single request looks malicious — it's a valid checkout, just duplicated

Detection relies on behavioral analysis (request rate, timing patterns) rather than
content inspection. A single legitimate checkout and a race condition attack send
identical HTTP requests.

### Mitigations

**Atomic database transactions:** Stock check and stock deduction execute as a
single indivisible operation. Either both succeed or neither does. No request can
slip between them.

**Final stock validation before commit:** Even if the check passed, re-validate
stock at the moment of committing the transaction. If another request already
deducted, abort.

**Idempotency keys:** Attach a unique key to each checkout request. If the same key
arrives twice, the second is rejected regardless of timing.

**Rate limiting / concurrency controls:** Block rapid repeated requests from the
same user or session. Limits the attack surface during high-traffic events.

---

## Challenges

Quick room. Race condition theory was already covered in Security+ so the concept
wasn't new. The practical gap was the specific Burp workflow — tab groups, duplicate
tabs, and the "Send group in parallel (last-byte sync)" option. That specific
mechanism wasn't something I'd used before. The distinction between last-byte sync
and regular parallel send is worth remembering: last-byte sync holds all requests
until fully prepared before releasing, which tightens the timing window further.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Race conditions,
TOCTOU vulnerabilities, application logic flaws, web application attacks.

**Domain 4.0 - Security Operations (28%):** Web application testing, Burp Suite,
concurrent request analysis, anomalous behavior detection.

---

## Evidence

![Burp Suite Parallel Group Setup](../07-Screenshots/Day20-1.png)
*15 duplicate POST /process_checkout requests grouped in Burp Repeater, ready for
parallel send with last-byte sync — attack group configured.*

![Negative Stock Result](../07-Screenshots/Day20-2.png)
*Orders page showing multiple confirmed SleighToy orders with stock pushed into
negative values — race condition successfully triggered.*

---

## Key Takeaways

**Three race condition types:**

| Type | What Happens |
|---|---|
| TOCTOU | Check and use are separate steps — another request lands between them |
| Shared resource | Multiple writes to same variable without locking — last write wins |
| Atomicity violation | Multi-step operation split across commits — concurrent request sees inconsistent state |

**Burp parallel attack workflow:**
```
1. FoxyProxy → Burp profile
2. Burp → Temporary project → Start Burp
3. Proxy → Intercept off
4. Make one legitimate request → capture POST /process_checkout
5. Send to Repeater → create tab group ("cart")
6. Duplicate tab × 15
7. Send dropdown → Send group in parallel (last-byte sync)
```

**Target details:**
- Login: `attacker / attacker@123`
- Product: SleighToy Limited Edition (10 units)
- Endpoint: `POST /process_checkout`
- Copies needed: 15 (enough to guarantee timing overlap)

**Blue Team detection signals:**
- Multiple identical POST requests from same session within milliseconds
- Inventory values going negative
- Order volume exceeding stock levels
- No content-level indicators — behavioral pattern only

**Mitigations:**

| Fix | How It Works |
|---|---|
| Atomic transactions | Check + deduct in single DB operation — no gap between them |
| Pre-commit revalidation | Re-check stock at commit time, abort if already depleted |
| Idempotency keys | Unique key per request — duplicate keys rejected outright |
| Rate limiting | Block rapid repeated requests from same session |

**Key terms:**

| Term | Definition |
|---|---|
| Race condition | Outcome depends on timing of concurrent operations on shared state |
| TOCTOU | Time of Check to Time of Use — vulnerability in the gap between validation and action |
| Atomicity | Property of an operation that executes as a single indivisible unit |
| Last-byte sync | Burp technique holding all parallel requests until fully ready, then firing simultaneously |
| Idempotency key | Unique token per request ensuring the same operation cannot be processed twice |
