# Day 16: Registry Forensics — Registry Furensics

**Date:** December 16, 2025  
**Time Spent:** 2 hours  
**Difficulty:** ★★★☆ *(Official rating: Easy — rated harder; new terminology, hive/root key mapping, and the binary data concept all hit at once)*  
**Category:** Digital Forensics / Windows Registry / Incident Response  
**Room:** https://tryhackme.com/room/registry-forensics-aoc2025-h6k9j2l5p8

---

## Overview

`dispatch-srv01` — TBFC's drone delivery coordinator — was compromised by King Malhare's
bandits. Abnormal activity started October 21, 2025. Using Registry Explorer, I
analyzed offline registry hives pulled from the system to identify what was installed,
where the attacker launched it from, and how it maintained persistence through reboots.

First time doing dedicated registry forensics. Used to tweak the registry for PC
optimization — this is a completely different application.

---

## What I Learned

### The Windows Registry

The registry is Windows' configuration database. It stores system settings, hardware
info, installed software, user preferences, startup programs, and a detailed record
of user activity. Everything Windows needs to run and everything an attacker does
leaves a trace here.

It is not stored in one file. It is split across multiple binary files called **hives**.

### Registry Hives

| Hive | Location | Contains |
|---|---|---|
| SYSTEM | `C:\Windows\System32\config\SYSTEM` | Services, boot config, drivers, hardware |
| SECURITY | `C:\Windows\System32\config\SECURITY` | Local security policies, audit settings |
| SOFTWARE | `C:\Windows\System32\config\SOFTWARE` | Installed programs, OS info, autostarts |
| SAM | `C:\Windows\System32\config\SAM` | User accounts, password hashes, group memberships |
| NTUSER.DAT | `C:\Users\[username]\NTUSER.DAT` | Recent files, user preferences, user autostarts |
| USRCLASS.DAT | `C:\Users\[username]\AppData\Local\Microsoft\Windows\USRCLASS.DAT` | Shellbags, jump lists |

Each hive stores more than what's listed above. These are the forensically relevant ones.

### Root Keys

Registry Editor does not show raw hive files. It organizes them under **Root Keys**,
which are structured containers mapped to the underlying hives.

| Root Key | Abbreviation | Contains |
|---|---|---|
| HKEY_LOCAL_MACHINE | HKLM | SYSTEM, SECURITY, SOFTWARE, SAM |
| HKEY_CURRENT_USER | HKCU | NTUSER.DAT (current user) |
| HKEY_USERS | HKU | All user NTUSER.DAT and USRCLASS.DAT profiles |
| HKEY_CLASSES_ROOT | HKCR | File associations — dynamically populated, no hive file |
| HKEY_CURRENT_CONFIG | HKCC | Hardware profile — dynamically populated, no hive file |

Forensic investigation paths almost always start with `HKLM\` (system-wide) or
`HKCU\` (user-specific). HKCR and HKCC don't have physical hive files on disk.

### Registry Editor vs. Registry Explorer

**Registry Editor (regedit):** Built into Windows. Cannot open offline hive files,
displays some values as raw binary, and any interaction on a live system risks
modifying evidence. Not suitable for forensic analysis.

**Registry Explorer:** Purpose-built forensic tool by Eric Zimmerman. Loads offline
hives, parses binary data into readable format, handles dirty hives via transaction
log replay, has built-in bookmarks to forensic key locations, and is read-only by
default. This is what you use for incident investigations.

### Loading Hives in Registry Explorer

```
1. Launch Registry Explorer (taskbar icon)
2. File → Load hive
3. Navigate to: C:\Users\Administrator\Desktop\Registry Hives
4. Hold SHIFT + click Open
   → Registry Explorer replays transaction logs automatically
   → Prompts confirm successful replay
5. Repeat for all hives
```

**Why SHIFT matters:** Hives collected from live systems can be "dirty" — they may
have incomplete write transactions. Holding SHIFT forces Registry Explorer to replay
the transaction logs, giving you a clean, consistent view of the data. Without it,
you may be analyzing incomplete state.

### Key Forensic Registry Paths

**Installed programs:**
```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
```

**Recently executed applications (GUI launches, with timestamps and run counts):**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist
```

**Startup persistence — system-wide:**
```
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

**Startup persistence — user-specific:**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

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
**Hive location:** `C:\Users\Administrator\Desktop\Registry Hives`

**Question 1: What application was installed?**

Key: `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` (SOFTWARE hive)  
Finding: **DroneManager Updater** — installed before October 21, 2025.

**Question 2: Where was it launched from?**

Key: `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist` (NTUSER.DAT)  
Finding: `C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe`

UserAssist logs GUI-launched applications with execution count and timestamp. The
path shows the attacker ran the installer directly from the Downloads folder.

**Question 3: How did it persist?**

Key: `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` (SOFTWARE hive)  
Finding: `"C:\Program Files\DroneManager\dronehelper.exe" --background`

The `--background` flag tells the process to run silently — no window, no visible
indicator to the user that something launched at login.

**Investigation workflow used:**
```
SOFTWARE hive → Uninstall key     → Identify suspicious app
NTUSER.DAT   → UserAssist key     → Find launch path and execution evidence
SOFTWARE hive → Run key           → Confirm persistence mechanism
```

---

## Challenges

The terminology hit hard all at once — hives, root keys, HKLM, HKCU, SIDs, dirty
hives, transaction logs. The conceptual gap between "hive files on disk" and "root
keys in Registry Editor" was the most confusing part. Once that mapping clicked,
everything else followed.

Knowing which hive to load for which question takes time to internalize. Installed
programs → SOFTWARE hive. User activity → NTUSER.DAT. It makes sense in retrospect
but isn't obvious on first contact.

The room is officially rated Easy. That's accurate once the structure clicks, but
hitting all of it simultaneously without prior registry forensics background made
it feel harder.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Malware persistence
mechanisms, post-exploitation techniques, infection vectors.

**Domain 4.0 - Security Operations (28%):** Incident response, digital forensics,
evidence collection and preservation, timeline reconstruction, system analysis.

---

## Evidence

![SOFTWARE Hive Uninstall Key](../07-Screenshots/Day16-1.png)
*DroneManager Updater identified in the Uninstall registry key, installed before
the October 21 compromise date.*

![UserAssist Launch Path](../07-Screenshots/Day16-2.png)
*UserAssist key showing DroneManager_Setup.exe launched from
C:\Users\dispatch.admin\Downloads — execution path and timestamp confirmed.*

---

## Key Takeaways

**Hive-to-Root Key mapping (memorize this):**

| Hive | Root Key |
|---|---|
| SYSTEM, SECURITY, SOFTWARE, SAM | `HKLM\` |
| NTUSER.DAT | `HKCU\` (current user) or `HKU\<SID>` |
| USRCLASS.DAT | `HKU\<SID>\Software\Classes` |

**Registry Explorer workflow:**
```
1. File → Load hive
2. Navigate to hive location
3. Hold SHIFT + Open (replays transaction logs)
4. Repeat for all hives
5. Use Available Bookmarks tab for fast navigation
6. Or search directly by key name
```

**Investigation cheat sheet:**

| Goal | Hive to Load | Key Path |
|---|---|---|
| Installed applications | SOFTWARE | `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` |
| GUI execution history | NTUSER.DAT | `HKCU\...\Explorer\UserAssist` |
| System-wide persistence | SOFTWARE | `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` |
| User-specific persistence | NTUSER.DAT | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` |
| Recently opened files | NTUSER.DAT | `HKCU\...\Explorer\RecentDocs` |
| Typed Explorer paths | NTUSER.DAT | `HKCU\...\Explorer\TypedPaths` |
| Explorer search terms | NTUSER.DAT | `HKCU\...\Explorer\WordWheelQuery` |
| USB device history | SYSTEM | `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR` |
| Hostname | SYSTEM | `HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName` |

**dispatch-srv01 findings (room answers):**

| Question | Answer |
|---|---|
| Application installed | DroneManager Updater |
| Launch path | `C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe` |
| Persistence value | `"C:\Program Files\DroneManager\dronehelper.exe" --background` |

**Red flags to hunt in Run keys:**
- Executables running from `C:\temp\`, `Downloads\`, or `AppData\`
- Unfamiliar application names with system-sounding values
- `--background`, `--hidden`, `-minimized` flags hiding execution
- Double file extensions (`update.pdf.exe`)
- Paths pointing outside `C:\Program Files\` or `C:\Windows\`
