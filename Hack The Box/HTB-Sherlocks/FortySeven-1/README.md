# FortySeven-1 — HTB Sherlock Write-up

**Date:** 2026-06-27  
**Prepared by:** uriel0byte  
**Difficulty:** Very Easy  
**Platform:** Hack The Box — Sherlocks  

---

## Executive Summary

This is a threat intelligence investigation, not a PCAP or log analysis. The work here is reading across three vendor reports and extracting accurate answers about an APT group called Mysterious Elephant (also tracked as APT-K-47 by Knownsec 404). The group has been active since at least 2022, targets government and diplomatic personnel in South Asia, and specializes in stealing WhatsApp data using a set of custom exfiltration tools. Their attack chains have evolved over time: early campaigns used borrowed downloaders and CHM lures; by 2025 they were deploying purpose-built loaders and reflective PE injection to stay off disk.

---

## Scenario

> *An APT group is using Hajj-themed phishing lures to target and steal WhatsApp data from government and diplomatic officials. Our team has gathered fragmented intelligence from public cybersecurity vendor reports, blog posts, and internal security alerts. Your task is to build a comprehensive profile of the threat actor responsible. You must connect the dots between different reports to answer questions about their identity, tools, and motives.*

---

## Intelligence Sources

| Source | Type | Coverage | Link |
|---|---|---|---|
| Evidence 1 | Securelist / Kaspersky GReAT | Latest 2025 campaign: BabShell, MemLoader HidenDesk, exfiltration toolset, file hashes | https://securelist.com/mysterious-elephant-apt-ttps-and-tools/117596/ |
| Evidence 2 | Knownsec 404 (Part 1) | ORPCBackdoor technical analysis, BITTER overlap, May 2023 campaigns | https://medium.com/@knownsec404team/apt-k-47-mysterious-elephant-a-new-apt-organization-in-south-asia-5c66f954477 |
| Evidence 3 | Knownsec 404 (Part 2) | Asyncshell evolution from TCP to HTTPS, January 2024 CVE-2023-38831 campaign, Hajj lure | https://medium.com/@knownsec404team/unveiling-the-past-and-present-of-apt-k-47-weapon-asyncshell-5a98f75c2d68 |

Reading all three before answering anything is the right approach. Each source covers a different time window and tool set. Evidence 1 is the most recent and has the IOC table. Evidence 3 is where the Hajj campaign and Asyncshell analysis live.

---

## Investigation

### Phase 1: Actor Identity

**Task 1 — What is the primary name of the APT group described in the SecureList report?**

> `Mysterious Elephant`

Kaspersky GReAT uses the name Mysterious Elephant. Knownsec 404 tracks the same group as APT-K-47. Both names refer to the same actor. The question asks for the Securelist name specifically, so Mysterious Elephant is the answer.

---

**Task 2 — According to the Knownsec 404 team's analysis (Evidence 3), since which year has this group's attack activity been dated back to?**

> `2022`

Evidence 3's overview section states APT-K-47's activity dates back to 2022. Evidence 1 says Kaspersky started tracking them in 2023. The question specifies Evidence 3, so 2022 is correct.

---

**Task 5 — The use of the backdoor links the APT to another well-known South Asian APT group. What is the name of this other group?**

> `bitter`

Evidence 2 documents that the ORPCBackdoor was found on network assets used by both the Confucius organization and BITTER. The ORPCBackdoor uses the `version.dll` template for DLL hijacking, a technique also associated with BITTER. The overlap in infrastructure and tooling is how Knownsec 404 makes the connection. HTB accepted the lowercase flag `bitter`; the source material refers to them as BITTER.

---

**Task 12 — In their early attack chains, Mysterious Elephant used a downloader that was previously associated with the Origami Elephant group. What was the name of this downloader?**

> `Vtyrei`

Evidence 1's emergence section covers the actor's early history. Mysterious Elephant started with tools borrowed from other APT groups, including the Vtyrei downloader, which Origami Elephant had used and abandoned. Mysterious Elephant picked it up, continued developing it, and eventually moved to their own custom tooling. This reuse pattern across South Asian APT groups is a recurring theme in this investigation.

---

### Phase 2: Tooling

**Task 3 — According to the Knownsec 404 team's analysis (Evidence 2), what is the name of the first malicious exported entry function of ORPCBackdoor?**

> `GetFileVersionInfoByHandleEx(void)`

Evidence 2 lists ORPCBackdoor's 17 export functions. Most of them are legitimate `version.dll` function names used to blend the DLL into the Windows file versioning library. The two malicious entries are `GetFileVersionInfoByHandleEx(void)` (first) and `DllEntryPoint` (second). The `void` in parentheses is part of the function signature and must be included in the answer.

---

**Task 4 — The previously mentioned backdoor checks for a file before creating persistence. What is the name of the file?**

> `ts.dat`

ORPCBackdoor checks whether `ts.dat` exists in its working directory before creating a scheduled task for persistence. If the file is already there, it skips persistence creation to avoid making duplicate entries. Once the task is created, it writes `ts.dat` as a marker. This is a standard persistence deduplication check.

---

**Task 6 — The APT group has consistently used and updated another backdoor since 2023, with its C2 communication evolving from TCP to HTTPS. What is the name of this tool?**

> `Asyncshell-v2`

Evidence 3 covers Asyncshell's development timeline. The original version communicated over raw TCP. By April 2024 Knownsec 404 found a new sample where the C2 communication had switched to HTTPS, which they designated Asyncshell-v2. The HTTPS version blends into normal browser traffic over port 443, making perimeter detection harder.

---

**Task 7 — To evade sandbox analysis, the MemLoader HidenDesk tool checks the number of active processes before running. What is the minimum number of processes required for it to proceed?**

> `40`

Evidence 1 documents MemLoader HidenDesk's sandbox evasion check: if fewer than 40 processes are running, it terminates. Sandboxes typically run minimal processes to keep the environment clean and avoid noise. A live user machine usually has well over 40 processes running at idle. The check is designed to fail silently in sandbox environments.

---

**Task 8 — The MemLoader HidenDesk tool creates a covert environment for its activities. What is the name of this hidden desktop?**

> `MalwareTech_Hidden`

After the process count check passes, MemLoader HidenDesk creates a hidden Windows desktop named `MalwareTech_Hidden` and switches to it. This technique comes from an open-source GitHub project. Running in a hidden desktop means the malware's activity won't be visible to the logged-in user. It then decrypts and reflectively loads the next stage payload (a Remcos RAT sample) into memory.

---

**Task 13 — In a January 2024 campaign delivering an Asyncshell payload, which CVE was exploited in the malicious archive file?**

> `CVE-2023-38831`

Evidence 3 covers this campaign. The attack chain: spear-phishing email → `AbroadDuty.zip` → CVE-2023-38831 WinRAR exploit → LNK file → `0.jpg.bat` → Asyncshell. CVE-2023-38831 is a critical WinRAR RCE (CVSS 7.8) affecting versions before 6.23. The exploit works by placing a malicious folder with the same name as a benign file inside the archive; when the user opens the benign file, WinRAR executes the hidden folder instead.

---

### Phase 3: Exfiltration & Framework Mapping

**Task 9 — What is the MITRE ATT&CK ID for the 'Registry Run Keys / Startup Folder' technique?**

> `T1547.001`

MemLoader HidenDesk places a shortcut in the startup folder to persist across reboots. This is Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder, sub-technique T1547.001 under the Persistence tactic.

---

**Task 10 — What is the name of the tool that recursively searches specific directories, including the "Desktop" and "Downloads" folders?**

> `Stom Exfiltrator`

Evidence 1 profiles three exfiltration tools: Uplo Exfiltrator (broad file type targeting, XOR-obfuscated C2 paths), Stom Exfiltrator (recursive directory search including Desktop, Downloads, all non-C drives, and a hardcoded WhatsApp AppData path), and ChromeStealer Exfiltrator (Chrome user data targeting). The directory traversal parameters in the question match Stom Exfiltrator specifically.

---

**Task 11 — What is the MITRE ATT&CK ID for the 'PowerShell' technique?**

> `T1059.001`

Evidence 1 notes Mysterious Elephant's heavy use of scripts for execution and payload delivery. PowerShell is Command and Scripting Interpreter: PowerShell, sub-technique T1059.001 under the Execution tactic.

---

**Task 14 — What is the MD5 hash of the ChromeStealer Exfiltrator sample named WhatsAppOB.exe?**

> `9e50adb6107067ff0bab73307f5499b6`

Located in the File Hashes table at the bottom of Evidence 1, under the ChromeStealer Exfiltrator section. The entry reads: `9e50adb6107067ff0bab73307f5499b6 WhatsAppOB.exe`. Always cross-check hashes character by character before submitting.

---

**Task 15 — What is the MITRE ATT&CK ID for the 'Exfiltration Over C2 Channel' technique?**

> `T1041`

All three exfiltration tools send stolen data back through the established C2 connection rather than opening a separate data channel. This is Exfiltration Over C2 Channel, technique T1041 under the Exfiltration tactic. Note: T1041 is a parent technique with no sub-techniques.

---

## MITRE ATT&CK Mapping

| Tactic | Technique ID | Name | Evidence |
|---|---|---|---|
| Execution | T1059.001 | Command and Scripting Interpreter: PowerShell | Scripts used extensively to stage and execute payloads |
| Persistence | T1547.001 | Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder | MemLoader HidenDesk places shortcut in startup folder |
| Exfiltration | T1041 | Exfiltration Over C2 Channel | All three exfiltrators upload stolen files through the established C2 connection |

---

## Lessons Learned

**CTI investigations are about reading carefully, not analyzing artifacts**

This Sherlock had no terminal, no Wireshark, no log files. The only skill being tested was whether I could read three reports accurately, understand what each one covered, and pull out the right detail for each question. I got Task 5 wrong initially because I was looking in Evidence 1 instead of Evidence 2. The source matters. Always read the question's evidence citation before opening anything.

**Three reports, three time windows — map them before reading**

Evidence 1 covers the 2025 campaign (BabShell, MemLoader, exfiltration tools). Evidence 2 covers May 2023 (ORPCBackdoor, BITTER overlap). Evidence 3 covers 2023 to mid-2024 (Asyncshell evolution, Hajj campaign, CVE-2023-38831). Understanding the timeline before answering individual questions prevents mixing up which tool or technique belongs to which campaign. I should have built this map on first read.

**Export tables tell you more than the malware's name**

The ORPCBackdoor uses legitimate `version.dll` function names to masquerade as a Windows system DLL. Every export except `GetFileVersionInfoByHandleEx(void)` and `DllEntryPoint` is a real Windows function. If you were only reading the DLL name or the imports, you'd miss the malicious entries entirely. The trick is DLL hijacking via white-and-black loading: a legitimate signed binary calls the malicious DLL thinking it's the real `version.dll`, and the malicious exports handle execution while the rest forward normally.

**The process count check is a standard sandbox evasion pattern**

The 40-process minimum in MemLoader HidenDesk is not unique to this group. Sandboxes run minimal processes; real machines don't. Checking the process count before executing is one of the simplest and most common evasion techniques. Any sample you analyze that terminates silently on first run in a sandbox is worth checking for this. Pair it with the hidden desktop technique (also well documented in open-source projects) and you have a payload that's invisible to both automated and manual analysis.

**Tool versioning is how APT groups respond to detection**

Asyncshell moved from TCP to HTTPS not because HTTPS is more capable, but because raw TCP C2 traffic is detectable by network defenders. Switching to port 443 with TLS means the C2 beacon looks identical to normal browser traffic at the perimeter. Attribution across tool versions requires tracking code logic and structural similarities, not just C2 addresses or file hashes, since those change with every campaign.

**Shared tooling between South Asian APT groups complicates attribution**

ORPCBackdoor's infrastructure overlapped with both Confucius and BITTER. Vtyrei was borrowed from Origami Elephant. This is common in the South Asian APT cluster: groups share code, reuse abandoned tools, and sometimes operate under unified infrastructure. Attribution based on a single indicator is unreliable. Multiple overlapping data points across infrastructure, tool behavior, targeting, and TTPs are what justify connecting two groups.

---

## Proof

Sherlock completion: https://labs.hackthebox.com/achievement/sherlock/2566537/1200

---

*Prepared by uriel0byte | github.com/uriel0byte*
