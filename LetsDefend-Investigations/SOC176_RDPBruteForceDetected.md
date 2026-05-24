# SOC176 - RDP Brute Force Detected

| Field          | Value                                      |
|----------------|--------------------------------------------|
| **Platform**   | LetsDefend                                 |
| **Alert ID**   | EventID 234                                |
| **Alert Time** | March 7, 2024 — 11:44 AM                  |
| **Category**   | Brute Force / External Remote Services     |
| **Verdict**    | True Positive — Host Compromised           |
| **Status**     | Closed                                     |

---

## Executive Summary

An external IP from China ran an RDP brute force against host `172.16.17.148` (Matthew) on March 7, 2024. The firewall allowed the traffic. After failing with generic account names, the attacker got in using the legitimate "Matthew" account. Post-compromise EDR analysis turned up discovery commands, a TightVNC installation for persistent remote access, and what looks like a Sticky Keys backdoor running as SYSTEM — neither of which were stopped before containment.

---

## Kill Chain

### 1. IP Reputation Check

Before touching the SIEM, I queried `218.92.0.56` across three sources.

| Source        | Finding                                                     |
|---------------|-------------------------------------------------------------|
| VirusTotal    | Flagged malicious. Origin: China. Tagged for SSH/RDP brute force activity. |
| AbuseIPDB     | High abuse confidence. Hundreds of thousands of reports. Same pattern: SSH/RDP brute force. |
| LetsDefend TI | Flagged malicious. Source: Anonymous.                       |

All three agree — this IP has a documented, long-running history of brute forcing remote access ports. No ambiguity about classification.

---

### 2. Initial Access

SIEM logs show 15 firewall events from `218.92.0.56` to `172.16.17.148` on Port 3389, all at 11:44 AM. The firewall action was **Allowed** — the port was open to the internet with nothing blocking it. Filtering for the attacker's source IP across all hosts returns only one destination: `172.16.17.148`. This was a targeted attack against Matthew specifically, not a subnet sweep.

---

### 3. Credential Access

Endpoint Security logs on the Windows 10 host confirm failed logon attempts starting at 11:44 AM.

- **Event ID 4625** — An account failed to log on
- **Error Code 0xC000006D** — Unknown username or bad password
- Usernames attempted: `admin`, `sysadmin`, `guest`

After cycling through generic names, the attacker authenticated as Matthew.

```
Username:    Matthew
EventID:     4624 (An account was successfully logged on)
Logon Type:  10 (RemoteInteractive)
Source IP:   218.92.0.56
```

`winlogon.exe` at `11:44:57` correlates with this logon. The host's last legitimate login was `04:00 AM` the same day — the attacker came in more than 7 hours later.

---

### 4. Discovery

After getting in, the attacker opened a command prompt and ran standard post-access recon. Timestamps from the Terminal History tab in Endpoint Security:

| Time     | Command                         | What it tells them                   |
|----------|---------------------------------|--------------------------------------|
| 11:45:18 | `C:\Windows\system32\cmd.exe`   | Opens a shell                        |
| 11:45:51 | `whoami`                        | Confirms current user and privilege  |
| 11:45:58 | `net user letsdefend`           | Enumerates local accounts            |
| 11:46:34 | `net localgroup administrators` | Checks who has admin rights          |
| 11:46:53 | `netstat -ano`                  | Maps active connections              |

Nothing exotic — but it confirms an interactive session with intent to go further.

---

### 5. Persistence

This is where the investigation went beyond the initial brute force alert. Checking the EDR process list and terminal history on the Matthew host turned up two persistence mechanisms the attacker put in place before containment.

**TightVNC (tvnserver.exe)**

Installed shortly after logon:

```
# Target Process Command Line :
"C:\Program Files\TightVNC\tvnserver.exe" -desktopserver
-logdir "C:\Windows\system32\config\systemprofile\AppData\Roaming\TightVNC"
-loglevel 0 -shmemname Global\heiejorlvibfsxlitbor

# Image Path :
C:\Program Files\TightVNC\tvnserver.exe

# Command Line :
"C:\Program Files\TightVNC\tvnserver.exe" -service
```

TightVNC gives GUI remote access that works independently of RDP. Even if Port 3389 had been blocked afterward, this would have kept the door open.

**Sticky Keys Backdoor (AtBroker.exe)**

`AtBroker.exe` (PID 4152) was spawned by `winlogon.exe` at `11:44:57 AM` — running as `NT AUTHORITY\SYSTEM`. Accessibility features like `sethc.exe` (Sticky Keys) can be redirected to drop a SYSTEM-level shell from the Windows login screen without any credentials. The SYSTEM process user here means the privilege escalation worked.

> `smss.exe` appears in the process tree as the parent of `winlogon.exe`. This is the normal Windows boot chain and is not an IOC — noted here for transparency.

---

## Containment & Remediation

**Containment**
Host `172.16.17.148` was isolated via the EDR platform, cutting the attacker's session and blocking lateral movement.

**Remediation**

- Reimage the host. A SYSTEM-level backdoor via Sticky Keys cannot be cleaned reliably — full OS reinstall is the only safe option.
- Reset the `Matthew` account and audit for any accounts created during the access window.
- Block `218.92.0.56` at the perimeter firewall.
- Close Port 3389 to the public internet. RDP should not be internet-facing under any circumstances.
- Enforce MFA on all remote access.
- Put remote access behind a VPN. RDP sessions should only be reachable from inside the tunnel.
- Remove or disable generic accounts (`admin`, `guest`, `sysadmin`) on internet-facing hosts.

---

## Indicators of Compromise (IOCs)

| Type                | Value                                 |
|---------------------|---------------------------------------|
| Malicious IP        | `218.92.0.56`                         |
| Compromised Account | `Matthew`                             |
| Attempted Username  | `admin`                               |
| Attempted Username  | `guest`                               |
| Attempted Username  | `sysadmin`                            |
| Malicious Process   | `tvnserver.exe` (TightVNC)            |
| Suspicious Binary   | `AtBroker.exe` (SYSTEM via winlogon)  |

---

## MITRE ATT&CK Mapping

| Tactic             | Technique                                                            |
|--------------------|----------------------------------------------------------------------|
| Initial Access     | T1133 — External Remote Services                                     |
| Credential Access  | T1110.001 — Brute Force: Password Guessing                           |
| Initial Access     | T1078.003 — Valid Accounts: Local Accounts                           |
| Discovery          | T1087.001 — Account Discovery: Local Account                         |
| Execution          | T1059.003 — Windows Command Shell                                    |
| Persistence        | T1546.008 — Accessibility Features (Sticky Keys / AtBroker)         |
| Persistence        | T1219 — Remote Access Software (TightVNC)                           |

---

*Written by: Supawat H. (uriel0byte) | LetsDefend SOC Practice*
