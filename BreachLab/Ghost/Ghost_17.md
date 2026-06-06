# BreachLab: Ghost - Level 17 → 18

## **Objective**

Execute remote commands via SSH to bypass a restricted environment that actively terminates interactive shell sessions.

## **Why this matters in the real world**

Restricted environments are everywhere in 2026: bastion hosts, container entrypoints, CI runners. Getting useful work done when someone has tried to lock down your shell is core to both attack and defense.

## **Investigation & Solution**

1. **Initial Connection & Denial:** Attempted a standard SSH login to the target user. The connection successfully authenticated, but immediately returned a "No interactive shell" banner and dropped the connection.

```bash
ssh ghost17@204.168.229.209 -p 2222
```

2. **Remote Command Execution (Reconnaissance):** To bypass the mechanism terminating the interactive session, I appended the `ls` command directly to the SSH connection string. This forced the server to execute the binary and return the output without spawning a full shell environment. The output revealed a file named `flag`.

```bash
ssh ghost17@204.168.229.209 -p 2222 ls
```

3. **Extraction:** I repeated the remote execution technique, this time passing the `cat` command targeted at the discovered file to extract the credential.

```bash
ssh ghost17@204.168.229.209 -p 2222 cat flag
```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1182213.svg)](https://asciinema.org/a/1182213)

## **Command Breakdown**

| **Command** | **Flag/Option** | **Purpose** |
| --- | --- | --- |
| `ssh` | `[command]` | Appending a command at the end of the SSH connection string executes it on the remote host directly, returning the standard output to the local terminal instead of spawning an interactive login shell. |

## **Notes & Takeaways**

**The Mechanics of the Trap:** The target system was configured to actively reject human operators. Defenders typically implement this in one of two ways:

1. Changing the user's default shell in `/etc/passwd` to `/sbin/nologin` or `/bin/false`.
2. Placing an `exit` command or a trap script inside the user's `.bashrc` or `.profile` files.

**Why Defenders Do This:** This is the standard, secure configuration for service accounts, bastion hosts (jump boxes), and automated CI/CD runners. These accounts require SSH access to transfer files (via SCP/SFTP) or trigger specific automated background tasks, but a human should never be allowed to log into them and explore the file system.

**The Bypass:** In this specific lab, the defender likely implemented the trap in the interactive shell profile (e.g., `.bashrc`). By appending the command directly to the SSH connection (`ssh user@ip ls`), the SSH daemon executes the requested binary directly. Because this does not spawn a full interactive TTY session, it bypasses the profile scripts that contain the trap, allowing the command to execute successfully.
