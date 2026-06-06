# BreachLab: Ghost - Level 18 → 19

## **Objective**

Locate and execute a binary with the SUID (Set Owner User ID) bit set to temporarily escalate privileges and retrieve the target credential.

## **Why this matters in the real world**

SUID is the single most common Linux privilege escalation path on earth. This is the first taste of what the Phantom track will cover in depth.

## **Investigation & Solution**

1. **Privilege Enumeration:** Instead of manually searching the file system, I utilized the `find` utility to systematically hunt for any executable files accessible by the `ghost18` group. I chained this with the `-exec` flag to automatically list the detailed permissions (`ls -la`) of any matches. After correcting a minor typo, the search surfaced a highly anomalous binary: `/usr/local/bin/ghost-reader`.

```bash
find /usr -group ghost18 -executable -exec ls -la {} +
```

2. **Target Analysis:** The `ls -la` output for `ghost-reader` showed the permissions `-rwsr-x---`. The crucial detail is the `s` in the owner's permission block, indicating the SUID bit is active, and that the file is owned by `ghost19`.

3. **Exploitation & Extraction:** I navigated to the binary's directory and executed it. Because of the SUID bit, the binary executed with the privileges of its owner (`ghost19`), allowing it to access and print the flag that `ghost18` would normally be denied from reading.

```bash
cd /usr/local/bin/
./ghost-reader
exit
```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1189488.svg)](https://asciinema.org/a/1189488)

## **Command Breakdown**

| **Command** | **Flag/Option** | **Purpose** |
| --- | --- | --- |
| `find` | `/usr` | Specifies the starting directory for the search. |
| `find` | `-group` | Filters for files belonging to the specified group (e.g., `ghost18`). |
| `find` | `-executable` | Filters for files that are executable by the current user. |
| `find` | `-exec` | Executes a specified command (like `ls -la`) on every file that matches the search criteria. The `{}` acts as a placeholder for the matched file, and `+` terminates the command. |

## **Notes & Takeaways**

**The Mechanics of SUID (Set-User-ID):**
Normally, when a user executes a program in Linux, the program runs with the privileges of the user who launched it. However, when the SUID bit is set on an executable file, the program runs with the privileges of the file's *owner*, regardless of who actually executed it.

In this lab, the `ghost-reader` binary was owned by `ghost19`. Even though `ghost18` launched the program, the SUID bit forced the operating system to execute it as if `ghost19` had run it. This allows a lower-privileged user to perform a highly specific, restricted task without needing the full password to the higher-level account.

**Spotting SUID:**
You can identify an SUID binary by looking at the standard file permissions. Instead of the standard `x` for executable in the owner's column, you will see an `s` (e.g., `-rwsr-x---`).

**The Defender's Dilemma:**
Administrators use SUID binaries so users can do things like change their own passwords (the `/usr/bin/passwd` command relies on SUID to write to the protected shadow file). However, if an administrator writes a custom script, assigns it SUID, and that script contains a vulnerability, an attacker can exploit it to execute arbitrary commands as `root`. Hunting for custom SUID binaries is a mandatory step in any Linux privilege escalation checklist.
