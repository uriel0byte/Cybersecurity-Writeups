# LetsDefend — SOC Investigation Reports

Hands-on SOC alert investigations completed on the LetsDefend platform. Each report documents the full investigation process: triage, threat intelligence, traffic and endpoint analysis, and remediation. 

---

## Investigations

| Alert ID | Rule Name                        | Category              | Verdict                        | Report |
|----------|----------------------------------|-----------------------|--------------------------------|--------|
| SOC176   | RDP Brute Force Detected         | Brute Force / External Remote Services | True Positive — Host Compromised | [View](SOC176_RDPBruteForceDetected.md) |
| SOC326   | Impersonating Domain MX Record Change Detected | Threat Intel / Phishing | True Positive — Host Compromised | [View](SOC326_ImpersonatingDomainMXRecordChangeDetected.md) |
| SOC282   | Phishing Alert - Deceptive Mail Detected | Phishing / Initial Access | True Positive — Host Compromised | [View](SOC282_DeceptiveMailDetected.md)
| SOC251 | Quishing Detected (QR Code Phishing) | Reconnaissance / Phishing | True Positive — Credentials Potentially Compromised | [View](SOC251_QuishingDetected.md) |
| SOC205 | Malicious Macro has been executed | Malware / Execution | True Positive — Host Compromised | [View](SOC205_MaliciousMacro.md) |
| SOC104 | Malware Detected | Malware / Execution | True Positive — Host Compromised | [View](SOC104_MalwareDetected.md) |
| SOC141 | Phishing URL Detected | Phishing / Initial Access | True Positive — Credentials Potentially Compromised | [View](SOC141_PhishingURLDetected.md) |
| SOC140 | Phishing Mail Detected - Suspicious Task Scheduler | Phishing / Initial Access | True Positive — Delivery Blocked | [View](SOC140_PhishingMailDetected.md) |
| SOC114 | Malicious Attachment Detected - Phishing Alert | Phishing / Execution | True Positive — Host Compromised | [View](SOC114_MaliciousAttachmentDetected.md) |

---

## Methodology

Each investigation follows the LetsDefend playbook structure but extends it where the evidence warrants. Standard workflow:

1. **Triage** — Review alert fields, classify source IP as internal or external.
2. **Threat Intelligence & IP Reputation** — Query VirusTotal, AbuseIPDB, and LetsDefend TI before touching logs.
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

[LetsDefend Profile](https://app.letsdefend.io/user/urielbyte)

---

*Supawat H. (uriel0byte) — Blue Team / SOC Tier 1 Practice*
