# BreachLab: Ghost - Level 20 → 21

## **Objective**

Identify a scheduled task (cron job) running as root and exploit a race condition in its execution to capture a protected credential.

## **Why this matters in the real world**

Cron is a privilege escalation gold mine and a persistence favorite. Every DFIR engineer checks cron directories in the first five minutes of a compromise investigation.

## **Investigation & Solution**

1. **Scheduled Task Enumeration:** I started by checking my own user's scheduled tasks (`crontab -l`), which returned empty. I then moved to enumerate system-wide cron drop-in directories and discovered a custom configuration file. Reading it revealed a script executing as `root` every minute.

```bash
crontab -l
ls /etc/cron.d
cat /etc/cron.d/ghost-level20
```


2. **Payload Analysis:** I reviewed the target script (`/opt/ghost-cron/job.sh`). The script reads a root-owned secret file and pipes it into a temporary file in the world-readable `/var/tmp/` directory. It then sleeps for two seconds before deleting the temporary file. This creates a two-second vulnerability window where the secret is exposed.

```bash
cat /opt/ghost-cron/job.sh
```


3. **Exploitation (Race Condition):** Instead of trying to manually catch the file during its brief existence, I wrote a bash "one-liner" to poll the directory. The infinite `while` loop continuously attempts to read the temporary file, silencing any "file not found" errors, and immediately breaks the loop the moment it successfully captures the flag.

```bash
while true; do cat /var/tmp/ghost-cron-output 2>/dev/null && break; done
exit
```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1200382.svg)](https://asciinema.org/a/1200382)

## **Command Breakdown**

| **Command** | **Flag/Option** | **Purpose** |
| --- | --- | --- |
| `crontab` | `-l` | Lists the current user's scheduled cron tasks. |
| `while true; do ... done` | None | Initiates an infinite loop in bash that will run until forcibly stopped or a `break` condition is met. |
| `2>/dev/null` | None | Redirects standard error (file descriptor 2) to `/dev/null` (the trash), silencing error messages in the terminal. |
| `&&` | None | Logical AND; executes the command on the right *only if* the command on the left succeeds. |

## **Notes & Takeaways**

**The Danger of /tmp and /var/tmp:**
Temporary directories are globally writable and readable by design. Any sensitive data written to these directories by a highly privileged account (like `root`) is instantly exposed to every other user on the system, even if it is only there for a fraction of a second.

**Exploiting the Window:**
This vulnerability is a simplified form of a Race Condition. The developer attempted to secure the process by deleting the file (`rm -f`), but the inclusion of `sleep 2` meant the execution was not atomic. Automating the read attempt using a `while` loop is the standard method for exploiting these brief windows of exposure, proving that security through rapid deletion is not security at all.
