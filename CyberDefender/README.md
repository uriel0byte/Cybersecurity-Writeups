# CyberDefenders — SOC Investigation Reports

Hands-on SOC investigations completed on the CyberDefenders platform. Each report covers the full analysis process: identifying the attack vector, tracing attacker behavior through logs or network traffic, and mapping findings to MITRE ATT&CK.

---

## Investigations

| Challenge | Category | Tools | Verdict | Report |
|---|---|---|---|---|
| Oski | Threat Intel / Malware Analysis | VirusTotal, ANY.RUN | True Positive — Stealc Infostealer Deployed | [View](Oski_Lab.md) |
| PoisonedCredentials | Network Forensics | Wireshark | True Positive — Credentials Compromised via LLMNR Poisoning | [View](PoisonedCredentials.md) |
| WebStrike | Network Forensics | Wireshark | True Positive — Web Shell Deployed, /etc/passwd Exfiltrated | [View](WebStrike_Lab.md) |

---

## Methodology

Each investigation is worked independently before reviewing any official solutions. Standard workflow varies by challenge category:

**Malware / Threat Intel**
1. Extract payload hash and query VirusTotal for classification and PE metadata.
2. Load sample in ANY.RUN sandbox — trace process tree, file drops, and network activity.
3. Pull C2 indicators from Relations and Behavior tabs.
4. Extract malware configuration details (encryption keys, staged DLLs).
5. Document IOCs, cleanup behavior, and exfiltration paths.

**Network Forensics**
1. Open PCAP in Wireshark — identify anomalous protocols or traffic patterns.
2. Apply targeted filters to isolate attacker activity (HTTP methods, NTLM auth, specific ports).
3. Follow HTTP/TCP streams to reconstruct attacker commands and payloads.
4. Identify affected hosts, compromised accounts, and exfiltrated data.
5. Document IOCs and map findings to MITRE ATT&CK.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Wireshark | PCAP analysis, protocol filtering, stream reconstruction |
| ANY.RUN | Interactive malware sandbox — process trees, network behavior, file drops |
| VirusTotal | File hash reputation, PE metadata, behavioral tags |

---

*Supawat H. (uriel0byte) — Blue Team / SOC Tier 1 Practice*
