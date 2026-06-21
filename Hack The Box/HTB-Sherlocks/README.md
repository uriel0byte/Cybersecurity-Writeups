# Hack The Box — Sherlocks Investigation Reports

Defensive security investigations completed via Hack The Box Sherlocks. Unlike traditional CTFs, these labs provide post-incident artifacts—PCAPs, Windows Event Logs, and disk images—requiring a full Incident Response approach.

The focus here is producing structured, technical write-ups that reconstruct the attack timeline and identify IOCs, directly supporting my focus on Blue Team operations and SOC analysis.

---

## Investigations

| Sherlock | Focus Area | Tools | Report |
| --- | --- | --- | --- |
| Brutus | DFIR | grep, file, wc, head, python3, utmp.py | [Link](Brutus/README.md) |
| Telly | SOC | Wireshark, DB Browser for SQLite, NVD (web) | [Link](Telly/README.md) |
| Noxious | SOC | Wireshark, hashcat | [Link](Noxious/README.md)

---

## Methodology

Each investigation is treated as a formal incident report. Standard workflow:

1. **Artifact Triage:** Verify file hashes and determine the scope of the provided evidence (e.g., network traffic vs. endpoint logs).
2. **Analysis:** Apply targeted filtering to network captures or parse system logs to isolate anomalous behavior. Focusing heavily on foundational networking and Linux command-line fluency, I prioritize reconstructing network streams and identifying unauthorized service executions.
3. **Attacker Attribution:** Map observed tactics and techniques to the MITRE ATT&CK framework.
4. **Documentation:** Detail the findings, exact commands used, and remediation steps. All platform flags are strictly redacted to comply with HTB rules and maintain focus on the methodology.

---
*Note: To comply with Hack The Box platform rules, all specific challenge flags within these reports have been redacted.*

*Supawat H. (uriel0byte) — Blue Team / SOC Tier 1 Practice*
