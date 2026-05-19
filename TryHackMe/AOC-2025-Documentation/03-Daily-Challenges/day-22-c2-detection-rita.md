# Day 22: C2 Detection with RITA — Command & Carol

**Date:** December 22, 2025  
**Time Spent:** 1 hour  
**Difficulty:** ★★★☆ *(Official rating: Medium — all tooling was new; RITA, Zeek, and behavioral C2 detection methodology hit simultaneously)*  
**Category:** Threat Hunting / Network Analysis / C2 Detection  
**Room:** https://tryhackme.com/room/detecting-c2-with-rita-aoc2025-m9n2b5v8c1

---

## Overview

After a series of attacks, TBFC went quiet. Too quiet. Rather than wait for the
next alert, Sir Elfo proposes proactive threat hunting — analyze the collected
network traffic and find any C2 beaconing that slipped through.

The task: convert two PCAP files to Zeek logs, import into RITA, then hunt for
active C2 communication to `malhare.net` and `rabbithole.malhare.net` using
behavioral analysis rather than signatures.

First time using both RITA and Zeek.

---

## What I Learned

### Command-and-Control (C2) Infrastructure

C2 is how attackers maintain communication with compromised systems after initial
access. The compromised host periodically contacts the attacker's server, receives
instructions, and reports back.

**The detection problem:** Modern C2 traffic is encrypted (HTTPS, DNS over HTTPS),
looks like normal web traffic on the surface, and uses legitimate ports. You cannot
just scan for known malicious strings. You need to analyze behavior.

**What behavioral C2 detection looks for:**

| Behavior | Description |
|---|---|
| Beaconing | Regular, periodic check-ins on predictable intervals |
| Long-lived connections | Sessions open for hours — interactive C2 |
| Low prevalence | Only 1-2 internal hosts contact a given external destination |
| Rare TLS signatures | Self-signed or unique certificates not seen elsewhere |
| Data exfiltration | High outbound byte volume to a single destination |
| DNS tunneling | Data encoded into DNS queries to bypass firewalls |

### Zeek — The Data Engine Behind RITA

RITA does not analyze raw PCAPs. It requires structured network logs, which Zeek
produces.

**What Zeek is:** An open-source Network Security Monitoring (NSM) tool that sits
on a SPAN port, network tap, or imported PCAP and converts raw packets into
structured log files. It is not an IDS/IPS — it does not block or alert. It
observes and records with high fidelity.

**Key Zeek log files:**

| Log | Contains |
|---|---|
| `conn.log` | All connections — src/dst IPs, ports, protocol, bytes, duration |
| `http.log` | HTTP requests — URIs, user agents, methods, status codes |
| `dns.log` | DNS queries and responses — domains, record types, answers |
| `ssl.log` | TLS sessions — certificates, ciphers, JA3 fingerprints |
| `files.log` | Files transferred — hashes, MIME types |

**PCAP → Zeek → RITA pipeline:**
```
Raw PCAP
    ↓
Zeek (parses protocols → structured logs)
    ↓
conn.log / http.log / dns.log / ssl.log
    ↓
RITA (behavioral analytics)
    ↓
C2 detections: beacons, tunneling, long connections, exfiltration
```

### RITA — Real Intelligence Threat Analytics

Open-source C2 detection framework by Active Countermeasures. Analyzes Zeek logs
using behavioral analytics and statistical models rather than signatures. Works
against encrypted traffic because it operates on metadata — timing, frequency,
volume, certificate patterns — not payload content.

**RITA analysis modules:**
- **SNI Connection Analysis** — TLS certificate patterns
- **IP Connection Analysis** — beaconing detection
- **DNS Analysis** — tunneling detection

**Why metadata beats signatures:** Even if the C2 payload is fully encrypted, RITA
still sees that one host made 1,337 connections to the same destination at 60-second
intervals with a consistent byte size and a rare TLS certificate seen nowhere else
in the dataset. That is high-confidence C2 detection without decrypting anything.

**Dataset size caveat:** Smaller datasets produce more false positives. A connection
that looks like beaconing in one hour of traffic may be normal software behavior
in a week of traffic. Always note the dataset window when interpreting results.

---

## Workflow — Commands

**Two PCAPs in this room:**
- `AsyncRAT.pcap` — tutorial PCAP, walked through first
- `rita_challenge.pcap` — the actual challenge

**Step 1: Convert PCAP to Zeek logs**
```bash
# Tutorial
zeek readpcap pcaps/AsyncRAT.pcap zeek_logs/asyncrat

# Challenge
zeek readpcap pcaps/rita_challenge.pcap zeek_logs/rita_challenge
```

**Step 2: Check Zeek output**
```bash
cd ~/zeek_logs/asyncrat && ls
# conn.log  dns.log  http.log  ssl.log  files.log
```

**Step 3: Import into RITA**
```bash
rita import --logs ~/zeek_logs/asyncrat/ --database asyncrat
rita import --logs ~/zeek_logs/rita_challenge/ --database rita_challenge
```

RITA parses the logs, normalizes data, runs SNI/IP/DNS analysis modules, applies
threat intelligence feeds (Feodo Tracker), and stores results in a named database.

**Step 4: Launch RITA UI**
```bash
rita view asyncrat
rita view rita_challenge
```

**Database management:**
```bash
rita list            # show all databases
rita delete asyncrat # remove database
```

---

## RITA Interface

**Results Pane** — all suspicious connections sorted by severity.

Key columns:

| Column | What to Look For |
|---|---|
| Score | Higher = more suspicious |
| Source IP | Internal compromised host |
| Destination | External C2 server or domain |
| Connection Count | Very high count = beaconing |
| Total Bytes | Very high = data exfiltration |
| Avg. Bytes | Consistent size = automated C2 |
| Port / Protocol | Non-standard port = investigate |
| Threat Modifiers | Why RITA flagged it |

**Details Pane** — click any entry to see connection metadata, beacon interval
analysis, TLS certificate details, JA3 hash, and per-connection byte breakdown.

**Activating search:** Press `/` to open the filter bar.

---

## RITA Threat Modifiers

Threat modifiers explain the specific behavioral reason a connection is flagged.
Multiple modifiers firing together = higher confidence.

**Prevalence**  
How many distinct internal hosts contact this external destination. Low number
(1-3 hosts) against a large internal network is suspicious — legitimate services
like Google or Microsoft would show hundreds of connecting hosts.

**Beacon Score**  
Measures regularity and consistency of connection timing. C2 malware checks in on
predictable schedules. High beacon score means the connection interval is too
regular to be human-driven traffic.

- 0–30% — likely benign
- 30–70% — worth investigating
- 70–100% — strong C2 indicator

**Long Connection**  
Sessions open for extended periods suggest interactive C2 rather than one-off
requests.

**Rare Signature**  
Unique TLS certificate or handshake pattern seen only in this dataset. Legitimate
services use well-known certificate authorities. Malware generates self-signed or
attacker-controlled certificates that stand out.

**First Seen**  
How recently the destination first appeared in the traffic. Newly contacted
destinations carry more risk than ones with months of history.

**MIME Type / URI Mismatch**  
Inconsistent HTTP headers — user-agent claims to be Chrome but the URI patterns
suggest a non-browser tool. Attackers fake headers to blend in.

---

## RITA Filter Syntax

```bash
# Press / to activate the filter bar

# Filter by destination
dst:rabbithole.malhare.net

# Filter by source
src:10.0.0.13

# Filter by beacon score
beacon:>=70
beacon:>=90

# Filter by severity
severity:>=8

# Sorting
sort:severity-desc
sort:duration-desc
sort:connections-desc
sort:bytes-desc

# Combined (the threat hunting query)
dst:rabbithole.malhare.net beacon:>=70 sort:duration-desc
```

---

## Challenge Investigation — rita_challenge.pcap

**Process:**
```bash
zeek readpcap pcaps/rita_challenge.pcap zeek_logs/rita_challenge
rita import --logs ~/zeek_logs/rita_challenge/ --database rita_challenge
rita view rita_challenge
```

**Room questions and answers:**

| Question | Answer |
|---|---|
| Which threat modifier shows how many hosts contact a destination? | Prevalence |
| How many hosts communicate with `malhare.net`? | Check Prevalence modifier in results for `dst:malhare.net` *(session-specific)* |
| Highest connection count to `rabbithole.malhare.net`? | Check top result after `dst:rabbithole.malhare.net sort:connections-desc` *(session-specific)* |
| Filter for `rabbithole.malhare.net`, beacon ≥70%, sorted by duration desc? | `dst:rabbithole.malhare.net beacon:>=70 sort:duration-desc` |
| Port used by `10.0.0.13` to connect to `rabbithole.malhare.net`? | Check Port column after `src:10.0.0.13 dst:rabbithole.malhare.net` *(session-specific)* |

**Note on session-specific answers:** Connection counts, prevalence numbers, and
ports are read directly from the RITA UI during the lab session. They are not
hardcoded — they come from the data in `rita_challenge.pcap`. The filter syntax
answers above are the ones that matter for future reference.

---

## Challenges

Everything was new: RITA, Zeek, the entire behavioral analysis methodology. The
only familiar element was PCAP files. The conceptual shift from signature-based
detection ("does this match a known bad pattern?") to behavioral detection
("does this connection's timing, frequency, and metadata look like C2?") took
time to internalize.

The threat modifier framework was the hardest part to absorb quickly. Understanding
why prevalence matters, why beacon score thresholds mean something specific, and
how to combine modifiers to build confidence — that's the core investigative skill
here and it only becomes intuitive through repetition.

RITA as a pivot point rather than a final answer was a key lesson. High severity
in RITA is a lead, not proof. The actual confirmation comes from digging into the
Zeek logs and PCAP.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** C2 infrastructure,
beaconing behavior, DNS tunneling, data exfiltration techniques.

**Domain 3.0 - Security Architecture (18%):** Network security monitoring, SPAN
ports, traffic collection architecture.

**Domain 4.0 - Security Operations (28%):** Threat hunting, behavioral analysis,
network forensics, log correlation, proactive defense.

---

## Evidence

![RITA Results Pane](../07-Screenshots/Day22-1.png)
*RITA results showing suspicious connections to malhare.net — beacon scores,
connection counts, and threat modifiers visible in the Results Pane.*

![RITA Filter Application](../07-Screenshots/Day22-2.png)
*Filter applied: `dst:rabbithole.malhare.net beacon:>=70 sort:duration-desc` —
targeted hunt for long-duration high-confidence C2 sessions.*

![Zeek Import Complete](../07-Screenshots/Day22-3.png)
*RITA import completion — SNI, IP, and DNS analysis modules all finished,
database ready for threat hunting.*

---

## Key Takeaways

**The three-command workflow:**
```bash
# 1. Convert
zeek readpcap pcaps/<file>.pcap zeek_logs/<output_dir>

# 2. Import
rita import --logs ~/zeek_logs/<dir>/ --database <name>

# 3. Hunt
rita view <name>
```

**Threat modifier priority (strongest C2 signal to weakest):**

| Priority | Modifier | Why It Matters |
|---|---|---|
| 1 | Beacon score ≥80% | Most reliable — too regular to be human |
| 2 | Low prevalence (1-3 hosts) | Targeted/unique contact |
| 3 | Rare signature | Custom attacker infrastructure |
| 4 | Long duration | Interactive session — not automated polling |
| 5 | High bytes sent | Exfiltration in progress |

**High-confidence C2 = multiple modifiers firing together.**

**Essential filter queries:**
```bash
# High-confidence beacons
beacon:>=70 sort:severity-desc

# Hunt exfiltration
sort:bytes-desc severity:>=7

# Specific domain investigation
dst:suspicious.com beacon:>=60 sort:duration-desc

# Low-prevalence connections (unique targets)
sort:prevalence-asc beacon:>=50
```

**Encrypted traffic still leaks C2 signals via:**
- Connection timing regularity (beacon score)
- TLS certificate uniqueness (rare signature)
- Consistent byte volume per connection
- Low host prevalence to the destination

**RITA as investigation pivot, not verdict:**
```
RITA high-severity entry
    ↓
Deep-dive Zeek logs (conn.log, http.log, dns.log)
    ↓
Extract PCAP segment → Wireshark
    ↓
Threat intel check (VirusTotal, AlienVault OTX)
    ↓
Endpoint correlation (EDR, Sysmon)
    ↓
Confirm or rule out
```

**Key terms:**

| Term | Definition |
|---|---|
| RITA | Real Intelligence Threat Analytics — open-source C2 detection framework by Active Countermeasures |
| Zeek | NSM tool converting raw packets to structured logs; not an IDS/IPS |
| Beaconing | Periodic, regular check-ins from compromised host to C2 server |
| Beacon score | RITA's measure of connection timing regularity (0–100%) |
| Prevalence | Number of distinct internal hosts contacting an external destination |
| Rare signature | TLS certificate or handshake pattern seen uniquely in the dataset |
| SPAN port | Switch port configured to copy traffic to a monitoring interface |
| JA3 hash | TLS fingerprint derived from handshake parameters |
| conn.log | Zeek log containing all connection metadata — the primary RITA input |
