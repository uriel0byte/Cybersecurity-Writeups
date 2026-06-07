# SOC282 — Phishing Alert - Deceptive Mail Detected

| Field | Value |
| --- | --- |
| **Platform** | LetsDefend |
| **Alert ID** | EventID 257 |
| **Alert Time** | May 13, 2024 — 09:22 AM |
| **Category** | Phishing / Initial Access |
| **Verdict** | True Positive — Host Compromised |
| **Status** | Closed |

---

## Executive Summary

An alert triggered for a deceptive email sent to Felix from `free@coffeeshooop.com` on May 13, 2024. The email gateway allowed the delivery. The email used a "Free Coffee Voucher" lure with a malicious embedded URL and a password-protected zip attachment. Endpoint analysis confirmed the user clicked the link, downloaded the zip, and executed the malicious payload (`Coffee.exe`). Post-compromise EDR logs showed the malware immediately spawning command prompts to run system discovery and establishing outbound network connections. The host was isolated and the email purged.

---

## Kill Chain

### 1. Threat Intelligence & IP Reputation

Before touching the internal logs, I checked the alert's SMTP IP address (`103.80.134.63`).

| Source | Result |
| --- | --- |
| LetsDefend TI | Flagged. Tagged: **phishing**. Source: Anonymous. |
| VirusTotal | Flagged malicious. Tagged for 'Phishing' originating from South Korea. |
| AbuseIPDB | 0 reports, but Whois data confirmed origin as South Korea (AS3786). |
| Cisco Talos | Flagged as Poor/Untrusted Reputation. |

The sender domain (`coffeeshooop.com`) had no reputation data, but the SMTP IP was definitively confirmed as malicious infrastructure.

![LetsDefend TI](./Screenshots/image_1780809819110_0.png)

---

### 2. Alert Verification

To verify the initial alert, I queried the SIEM (Log Management) for the source IP `103.80.134.63`. Filtering the date range from May 13, 2024, to May 31, 2024, returned exactly one hit. The raw log confirmed an Exchange event matching the sender `free@coffeeshooop.com` and the destination `Felix@letsdefend.io`. The action was marked as **Allowed**, verifying that this was a true event and the malicious email successfully bypassed the perimeter filters.

![SIEM Event Verification](./Screenshots/image_1780810732589_0.png)

---

### 3. Email Analysis

I pivoted to Email Security to find the specific email delivered to Felix.

| Field | Value |
| --- | --- |
| Sender | `free@coffeeshooop.com` |
| Recipient | `Felix@letsdefend.io` |
| Subject | Free Coffee Voucher |
| Action | Allowed |

![Email Security Gateway Search](./Screenshots/image_1780811104948_0.png)

![Phishing Email](./Screenshots/image_1780811162484_0.png)

![Attachment](./Screenshots/image_1780811175905_0.png)

**Why this email is suspicious:**

* The sender domain is a clear typosquat (`coffeeshooop` with three 'o's).
* The email relies on a false sense of urgency ("Hurry, this offer expires soon!").
* The "Redeem Now" button redirects through `cyberlearn.academy` to directly download a file from an AWS S3 bucket (`[files-ld.s3.us-east-2.amazonaws.com/.../free-coffee.zip](https://files-ld.s3.us-east-2.amazonaws.com/.../free-coffee.zip)`).
* The email includes a file attachment (`free-coffee.zip`) and provides the password "infected" in plaintext. This is a common evasion technique to prevent email gateways from scanning the zip file contents.

---

### 4. Endpoint Analysis

Because the firewall action was **Allowed**, I checked Felix's host (`172.16.20.151`) in Endpoint Security to see if the payload was detonated.

![Host Containment & Browser History](./Screenshots/image_1780811917101_0.png)

* **Browser History:** At `12:59` (about 3.5 hours after delivery), Felix visited the AWS S3 URL, confirming the link was clicked and the file was downloaded.

* **Processes History:** At `13:00:38`, `Coffee.exe` (PID 56697) was executed from `C:\Users\Felix\Downloads\Coffee.exe`.

![Processes History](./Screenshots/image_1780812725401_0.png)

![Processes History](./Screenshots/image_1780812846657_0.png)

* **Terminal History:** This is where the compromise was confirmed. Starting at `13:01`, `Coffee.exe` spawned `cmd.exe` and rapidly ran a sequence of automated discovery commands:

  * `systeminfo`
  * `hostname`
  * `wmic logicaldisk get caption,description...`
  * `net user`
  * `tasklist /svc`
  * `ipconfig /all`
  * `route print`

![cmd.exe spawned by Coffee.exe](./Screenshots/image_1780812823062_0.png)

![Terminal History](./Screenshots/image_1780812218299_0.png)

* **Network Action:** Following the execution, the host initiated several outbound connections, indicating potential Command and Control (C2) activity.

![Network Action](./Screenshots/image_1780812535579_0.png)

---

## Containment & Remediation

**Containment**
* Host `172.16.20.151` (Felix) was immediately isolated via the EDR platform to sever any active connections and prevent lateral movement.
* The phishing email was permanently deleted from Felix's inbox via the Email Security Gateway to prevent further interaction.

**Remediation**
* Escalate to Tier 2 for full endpoint forensics and reimaging. The local discovery commands prove the malware successfully detonated.
* Force a password reset for Felix's account.
* Block the malicious SMTP IP (`103.80.134.63`) and sender domain (`coffeeshooop.com`) at the perimeter firewall.
* Block the malicious AWS S3 URL at the web proxy.

---

## Indicators of Compromise (IOCs)

| Type | Value |
| --- | --- |
| Malicious SMTP IP | `103.80.134.63` |
| Phishing Domain | `coffeeshooop.com` |
| Sender Address | `free@coffeeshooop.com` |
| Payload URL | `https://download.cyberlearn.academy/download/download?url=https://files-ld.s3.us-east-2.amazonaws.com/59cbd215-76ea-434d-93ca-4d6aec3bac98-free-coffee.zip ` |
| Malicious Hash (SHA256) | `CD903AD2211CF7D166646D75E57FB866000F4A3B870B5EC759929BE2FD81D334` (Coffee.exe) |

---

## MITRE ATT&CK Mapping

| Tactic | Technique |
| --- | --- |
| Initial Access | T1566.002 — Phishing: Spearphishing Link |
| Execution | T1204.002 — User Execution: Malicious File |
| Execution | T1059.003 — Command and Scripting Interpreter: Windows Command Shell |
| Discovery | T1087.001 — Account Discovery: Local Account |
| Discovery | T1082 — System Information Discovery |

---

![Score](./Screenshots/PhishingAlert_LetsDefend.png)

---

*Written by: Supawat H. (uriel0byte) | LetsDefend SOC Practice*
