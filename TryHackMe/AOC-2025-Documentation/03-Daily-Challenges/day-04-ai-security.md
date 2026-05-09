# Day 4: old sAInt nick - AI in Security

**Date:** December 4, 2025  
**Time Spent:** 1 hour  
**Difficulty:** ★☆☆☆  
**Category:** AI Security / Emerging Technologies  
**Room:** https://tryhackme.com/room/AIforcyber-aoc2025-y9wWQ1zRgB

---

## Overview

TBFC replaced their outdated chatbot Van Chatty with a new AI assistant called Van
SolveIT. The room walked through three showcases demonstrating where AI fits into
security workflows: Red Team exploit generation, Blue Team log analysis, and software
code scanning. Straightforward conceptual day with guided interaction throughout.

---

## What I Learned

### Why AI Matters in Security Now

AI has moved from novelty to embedded workflow tool. Organizations want to see
experience with it, not avoidance. The value is in handling high-volume, repetitive
tasks so analysts can focus on work that actually requires human judgment.

**Three things AI does well in security:**

| Capability | Application |
|---|---|
| Processing large data volumes | Correlating logs, network flows, and endpoint telemetry simultaneously |
| Behavior analysis | Flagging deviations from established baselines |
| Generative AI | Summarizing alert chains, generating incident context |

### Blue Team Applications

**Alert triage:**  
AI continuously processes logs, network flows, and endpoint signals. It adds context
to alerts and filters noise before an analyst ever sees them. Vendors have built this
into firewalls, IDS platforms, and SIEM products.

**Automated response examples:**
- Isolating infected devices from the network
- Blocking suspicious email senders
- Flagging impossible travel login attempts
- Quarantining files detected by EDR

**Blue Team showcase:** Van SolveIT analyzed web server logs, identified attack
patterns including SQL injection and directory traversal, summarized findings, and
suggested remediation steps.

### Red Team Applications

Reconnaissance traditionally consumes 60-70% of a pentest engagement. AI can automate
most of it — OSINT collection, subdomain enumeration, Nmap output analysis, technology
stack identification — freeing the tester to focus on actual exploitation.

**Red Team showcase:** Van SolveIT generated a Python exploit script targeting a
vulnerable web application at `MACHINE_IP:5000`. Updated the IP placeholder, executed
the script, captured the flag from output.

**Important caveat:** AI-generated exploits need human review before execution. A
script that causes a race condition or accidentally floods a production service is a
real outcome, not a hypothetical one. Human oversight is not optional here.

### Software Security Applications

AI works well as a code auditing assistant. It can scan for SQL injection, XSS,
hardcoded credentials, path traversal, and insecure deserialization without executing
the code (SAST), or test a running application for runtime flaws (DAST).

The irony worth remembering: AI is good at finding vulnerabilities in code but not
reliable at writing secure code. It was trained on internet data, which includes plenty
of vulnerable examples. "Vibe coding" with AI introduces security debt.

Use AI to audit. Have a human verify before anything goes to production.

**Software showcase:** Van SolveIT scanned source code, identified vulnerable sections,
explained the vulnerability types, and recommended fixes.

### Limitations Worth Knowing

**Accuracy is not guaranteed.** False positives waste analyst time. False negatives
mean missed attacks. Always verify output.

**Data privacy.** Public AI services may log inputs. Never feed sensitive client data,
credentials, network architecture, or proprietary source code into a public model.

**Novel threats.** AI is trained on past patterns. Zero-days and genuinely new attack
techniques fall outside that training. Human expertise covers what AI cannot.

**Black box problem.** Explaining why AI flagged something to a client or stakeholder
is difficult when the model cannot articulate its own reasoning.

**Model security.** AI itself can be attacked — adversarial inputs, training data
poisoning, model extraction via repeated queries.

---

## Challenges

Nothing technical here. The only minor friction was response latency — Van SolveIT
took 1-2 minutes to generate replies. The room flags this upfront so it did not cause
confusion.

---

## Security+ Alignment

**Domain 1.0 - General Security Concepts (12%):** Security automation, data privacy
controls.

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Emerging AI-based
threats, adversarial machine learning, vulnerabilities introduced by AI systems.

**Domain 4.0 - Security Operations (28%):** AI-assisted threat detection, SOC
automation, automated response mechanisms.

---

## Key Takeaways

**Where AI helps:**
- High-volume data processing (logs, telemetry, network flows)
- Repetitive tasks (alert triage, OSINT collection)
- Pattern recognition across large datasets
- Code vulnerability scanning (SAST/DAST)
- Threat intelligence summarization

**Where AI needs human oversight:**
- Final security decisions
- Complex investigations requiring context
- Novel or zero-day threats
- Penetration test execution (damage risk)
- Production code generation
- Compliance and legal decisions

**Data privacy rule:** Never input sensitive data into public AI models. This includes
client network architecture, vulnerability details, credentials, PII, and proprietary
source code.

**Van SolveIT workflow summary:**

```
Stage 1 - Red Team:
Prompt AI → receive exploit script → update IP placeholder → execute → capture flag

Stage 2 - Blue Team:
Provide logs → AI identifies patterns → reviews findings → suggests remediation

Stage 3 - Software:
Submit code → AI SAST scan → vulnerability report → recommended fixes
```

**Commercial products that mirror this:**

| Category | Examples |
|---|---|
| SIEM with AI | Splunk ES, Microsoft Sentinel, IBM QRadar |
| EDR with AI | CrowdStrike Falcon, SentinelOne |
| Network AI | Darktrace, Vectra AI |
| Code security | Snyk, Veracode, Checkmarx |
| SOAR | Palo Alto Cortex XSOAR, Splunk SOAR |
