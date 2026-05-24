# LetsDefend — SOC Investigation Reports

Hands-on SOC alert investigations completed on the LetsDefend platform. Each report documents the full investigation process: triage, threat intelligence, traffic and endpoint analysis, and remediation. 

---

## Investigations

| Alert ID | Rule Name                        | Category              | Verdict                        | Report |
|----------|----------------------------------|-----------------------|--------------------------------|--------|
| SOC176   | RDP Brute Force Detected         | Brute Force           | True Positive — Host Compromised | [View](SOC176_RDPBruteForceDetected.md) |

---

## Methodology

Each investigation follows the LetsDefend playbook structure but extends it where the evidence warrants. Standard workflow:

1. **Triage** — Review alert fields, classify source IP as internal or external.
2. **IP Reputation** — Query VirusTotal, AbuseIPDB, and LetsDefend TI before touching logs.
3. **Traffic Analysis** — Confirm attack scope via SIEM log management. Single host or network-wide?
4. **Endpoint Analysis** — Check Windows Event Logs (4624/4625), process list, terminal history, and browser history on the affected host via Endpoint Security.
5. **Verdict & Containment** — Isolate if compromised. Document findings, IOCs, and remediation steps.
6. **MITRE ATT&CK Mapping** — Map confirmed techniques to ATT&CK sub-techniques where possible.

*Each report is written from my own investigation first.*
*The official LetsDefend report is reviewed afterward as a calibration check.*

---

## Tools Used

| Tool             | Purpose                                      |
|------------------|----------------------------------------------|
| LetsDefend SIEM  | Log management and firewall event analysis   |
| LetsDefend EDR   | Endpoint process, terminal, and network analysis |
| VirusTotal       | IP and file reputation                       |
| AbuseIPDB        | IP abuse history and confidence scoring      |
| LetsDefend TI    | Internal threat intelligence feed            |

---

*Supawat H. (uriel0byte) — Blue Team / SOC Tier 1 Practice*
