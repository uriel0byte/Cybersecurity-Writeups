# Day 16: Registry Forensics — Registry Furensics

**Date:** December 16, 2025  
**Time Spent:** 2 hours  
**Difficulty:** ★★★☆ *(Official rating: Easy — rated harder; new terminology, hive/root key mapping, and binary data concepts all hit at once)*  
**Category:** Digital Forensics / Windows Registry / Incident Response  
**Room:** https://tryhackme.com/room/registry-forensics-aoc2025-h6k9j2l5p8

---

## Overview

`dispatch-srv01` — TBFC's drone delivery coordinator — was compromised by King
Malhare's bandits. Abnormal activity started October 21, 2025. Using Registry
Explorer, I analyzed offline registry hives pulled from the system to identify what
was installed, where the attacker launched it from, and how it maintained persistence
through reboots.

First time doing dedicated registry forensics. Used to tweak the registry for PC
optimization — this is a completely different application of the same tool.

---

## What I Learned

### The Windows Registry

The registry is Windows' configuration database. It stores system settings, hardware
info, installed software, user preferences, startup programs, and a detailed record
of user activity. Everything Windows needs to function and everything a user or
attacker does leaves a trace here.

It is not a single file. It is split across multiple binary files called **hives**,
each storing a different category of configuration data. Because these files are
binary, you cannot open them in Notepad or any standard text editor — you get
unreadable garbage. They require a tool that knows how to parse the format.

### Registry Hives

| Hive | Location | Contains |
|---|---|---|
| SYSTEM | `C:\Windows\System32\config\SYSTEM` | Services, boot config, drivers, hardware, mounted devices |
| SECURITY | `C:\Windows\System32\config\SECURITY` | Local security policies, audit settings |
| SOFTWARE | `C:\Windows\System32\config\SOFTWARE` | Installed programs, OS info, autostarts, program settings |
| SAM | `C:\Windows\System32\config\SAM` | User accounts, password hashes, group memberships, account statuses |
| NTUSER.DAT | `C:\Users\[username]\NTUSER.DAT` | Recent files, user preferences, user-specific autostarts |
| USRCLASS.DAT | `C:\Users\[username]\AppData\Local\Microsoft\Windows\USRCLASS.DAT` | Shellbags, jump lists |

**USRCLASS.DAT note:** Shellbags store records of folders a user has opened and
navigated — including folders on USB drives, network shares, and deleted directories.
Jump lists store recently accessed files per application. Both are useful for proving
a user accessed specific locations, even after files are deleted.

Each hive stores more than the examples above. These are the forensically relevant
highlights.

### Root Keys

Registry Editor does not expose raw hive files directly. It organizes them into
**Root Keys** — structured logical containers mapped to the underlying physical hives.

| Root Key | Abbreviation | Backed By |
|---|---|---|
| HKEY_LOCAL_MACHINE | HKLM | SYSTEM, SECURITY, SOFTWARE, SAM |
| HKEY_CURRENT_USER | HKCU | NTUSER.DAT (currently logged-in user) |
| HKEY_USERS | HKU | All user NTUSER.DAT and USRCLASS.DAT profiles |
| HKEY_CLASSES_ROOT | HKCR | File associations — dynamically populated, no hive file |
| HKEY_CURRENT_CONFIG | HKCC | Hardware profile — dynamically populated, no hive file |

HKCR and HKCC are not backed by separate hive files on disk. Windows populates them
at runtime from data spread across other hives. You will not find a HKCR or HKCC
file in `C:\Windows\System32\config\`.

Nearly all forensic investigation paths start with `HKLM\` (system-wide data) or
`HKCU\` (user-specific data).

### Registry Editor vs. Registry Explorer

**Registry Editor (regedit):** Windows built-in. Cannot open offline hive files,
renders some values as raw unreadable binary, and any interaction on a live system
risks modifying evidence. Not suitable for forensic work.

**Registry Explorer:** Purpose-built forensic tool by Eric Zimmerman. Loads offline
hives, parses binary into readable output, handles dirty hives through transaction
log replay, has built-in bookmarks to key forensic locations, and is read-only by
default. This is what you use for incident response.

### Dirty Hives and Transaction Logs

Hives collected from live running systems can be "dirty" — they may have incomplete
write transactions that were in progress when the hive was copied. Analyzing a dirty
hive means potentially working with inconsistent or partial data.

Registry Explorer solves this by replaying the associated transaction log files
alongside the hive, which reconstructs a clean, consistent state before analysis.

**How to load correctly:**
```
1. File → Load hive
2. Navigate to hive location
3. Hold SHIFT + click Open
   → Transaction logs replay automatically
   → Prompts confirm successful replay
4. Repeat for each hive
```

Skipping the SHIFT step means you may be analyzing a hive with missing or corrupted
entries. Always load with transaction logs.

### Key Forensic Registry Paths

**Installed programs:**
```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
```

**Recently executed applications via GUI (with timestamps and run counts):**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist
```

Note: UserAssist encodes application paths using ROT-13, so the paths appear
scrambled in the raw data. Registry Explorer decodes these automatically. The
key stores execution count and focus time per application alongside the path.

**Commands typed in the Run dialog (Win+R):**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```

**Startup persistence — system-wide (runs on any user login):**
```
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

**Startup persistence — user-specific:**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

**One-time startup — runs once then deletes itself:**
```
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

Run vs. RunOnce: `Run` entries survive and re-execute every boot. `RunOnce` entries
execute once and are automatically deleted afterward. Malware installers sometimes
use RunOnce for setup steps; persistent backdoors use Run.

**Recently accessed files:**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
```

**Paths typed in Explorer address bar:**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
```

**Search terms typed in Explorer:**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery
```

**Application install paths:**
```
HKLM\Software\Microsoft\Windows\CurrentVersion\App Paths
```

**Computer hostname:**
```
HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName
```

**Connected USB devices:**
```
HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR
```

### Attack Investigation — dispatch-srv01

**Abnormal activity started:** October 21, 2025  
**Hive location on machine:** `C:\Users\Administrator\Desktop\Registry Hives`

**Question 1: What application was installed?**

Key: `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` (SOFTWARE hive)  
Finding: **DroneManager Updater** — installed before October 21, 2025.

"Updater" naming is a common social engineering disguise. Users expect update tools
to run silently and regularly, making them less likely to question the process.

**Question 2: Where was it launched from?**

Key: `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist` (NTUSER.DAT)  
Finding: `C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe`

Launched directly from the Downloads folder — the standard indicator of a
user downloading and immediately executing a file from the internet. Running
installers from Downloads rather than a controlled software deployment path
is a red flag in any corporate environment.

**Question 3: How did it persist?**

Key: `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` (SOFTWARE hive)  
Finding: `"C:\Program Files\DroneManager\dronehelper.exe" --background`

The `--background` flag suppresses any visible window, so the process starts
silently at every login with no indication to the user that anything launched.

**Full attack chain:**
```
User downloads DroneManager_Setup.exe from internet
        ↓
Installer executed from Downloads folder
(C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe)
        ↓
Malware installs to C:\Program Files\DroneManager\
        ↓
Run key modified — dronehelper.exe added with --background flag
        ↓
dronehelper.exe silently executes on every user login
        ↓
Persistent access maintained on dispatch-srv01
```

**Investigation workflow used:**
```
SOFTWARE hive → Uninstall key   → Identify suspicious app + install date
NTUSER.DAT   → UserAssist key   → Confirm execution path and timestamp
SOFTWARE hive → Run key         → Confirm persistence mechanism
```

---

## Challenges

The terminology was the first wall — hives, root keys, HKLM, HKCU, HKU, SIDs,
dirty hives, transaction logs, all arriving together. The conceptual gap between
"physical hive files on disk" and "root keys displayed in Registry Editor" took
time to resolve. Once that mapping clicked, navigation became logical.

Knowing which hive to load for which question is something that needs to become
instinct. Installed programs → SOFTWARE. User execution history → NTUSER.DAT.
Obvious in retrospect, not obvious going in cold.

The room is officially rated Easy. That makes sense once the structure settles,
but hitting the full hive/root key/binary data model for the first time without
prior registry forensics exposure made it sit harder than the label suggests.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Malware persistence
mechanisms, post-exploitation techniques, infection vectors.

**Domain 4.0 - Security Operations (28%):** Incident response, digital forensics,
evidence collection and preservation, timeline reconstruction, system analysis.

---

## Evidence

![SOFTWARE Hive Uninstall Key](../07-Screenshots/Day16-1.png)
*DroneManager Updater identified in the Uninstall key — installation date confirmed
before October 21 compromise date.*

![Persistence Run Key](../07-Screenshots/Day16-2.png)
*Run key entry showing dronehelper.exe with --background flag — persistence
mechanism confirmed in SOFTWARE hive.*

---

## Key Takeaways

**Hive-to-Root Key mapping:**

| Hive | Root Key Location |
|---|---|
| SYSTEM | `HKLM\SYSTEM` |
| SECURITY | `HKLM\SECURITY` |
| SOFTWARE | `HKLM\SOFTWARE` |
| SAM | `HKLM\SAM` |
| NTUSER.DAT | `HKCU\` (current user) or `HKU\<SID>` |
| USRCLASS.DAT | `HKU\<SID>\Software\Classes` |

**Registry Explorer load procedure:**
```
1. File → Load hive
2. Navigate to hive folder
3. Hold SHIFT + click Open  ← mandatory for dirty hives
4. Confirm transaction log replay prompts
5. Repeat for all hives
6. Use Available Bookmarks tab for fast forensic key navigation
```

**Investigation cheat sheet:**

| Goal | Hive | Key Path |
|---|---|---|
| Installed applications | SOFTWARE | `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` |
| GUI execution history | NTUSER.DAT | `HKCU\...\Explorer\UserAssist` |
| Run dialog history | NTUSER.DAT | `HKCU\...\Explorer\RunMRU` |
| System-wide persistence | SOFTWARE | `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` |
| User-specific persistence | NTUSER.DAT | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` |
| One-time startup | SOFTWARE / NTUSER.DAT | `...\CurrentVersion\RunOnce` |
| Recently opened files | NTUSER.DAT | `HKCU\...\Explorer\RecentDocs` |
| Typed Explorer paths | NTUSER.DAT | `HKCU\...\Explorer\TypedPaths` |
| Explorer search terms | NTUSER.DAT | `HKCU\...\Explorer\WordWheelQuery` |
| USB device history | SYSTEM | `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR` |
| Hostname | SYSTEM | `HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName` |

**dispatch-srv01 room answers:**

| Question | Answer |
|---|---|
| Application installed | DroneManager Updater |
| Launch path | `C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe` |
| Persistence value | `"C:\Program Files\DroneManager\dronehelper.exe" --background` |

**Red flags to hunt in Run keys:**
- Executables in `C:\temp\`, `Downloads\`, or `AppData\` (not Program Files)
- Unfamiliar names with system-sounding values (`UpdateService`, `WindowsHelper`)
- Silent execution flags: `--background`, `--hidden`, `-minimized`, `-silent`
- Double extensions: `update.pdf.exe`
- Entries created at suspicious times (outside business hours, during known incident window)

**Key terms:**

| Term | Definition |
|---|---|
| Hive | Physical binary file on disk storing a portion of the registry |
| Root Key | Logical container in Registry Editor mapping to one or more hives |
| Dirty hive | Hive collected from a live system with potentially incomplete transactions |
| Transaction log | Companion file that records uncommitted hive changes for replay |
| UserAssist | Registry key logging GUI-launched applications; paths are ROT-13 encoded |
| RunMRU | Registry key storing history of commands typed in the Win+R Run dialog |
| Shellbag | USRCLASS.DAT artifact recording folder navigation history |
| SAM | Security Accounts Manager hive; stores local user accounts and password hashes |
