# BreachLab: Ghost - Level 8 → Level 9

## **Objective**

Identify a suspicious background process and interrogate the `/proc` pseudo-filesystem to extract hidden credentials from live memory.

## **Why this matters in the real world**

Fileless malware analysis and live incident response. This is what an IR engineer does at 3am when a box is already compromised and disk forensics is too slow.

## **Investigation & Solution**

1.  **Process Enumeration:** Initialized the investigation by listing all currently running processes using `ps aux`. Scanned the output for anomalies and identified two suspicious Python scripts running under the `ghost8` user (`level8-daemon.py`).

    ```Bash
    ps aux
    ps -q 41 aux
    ps -q 42 aux
    ```
    
2.  **Source Code Inspection:** Read the contents of the Python script to understand its function. The script was a simple infinite loop utilizing `time.sleep(3600)`, confirming it was a decoy meant to hold a process open without consuming CPU cycles.

    ```Bash
    cat /usr/local/bin/level8-daemon.py
    ```
    
3.  **Kernel-Level Interrogation:** To find what the script was hiding in memory, I moved past standard user-space tools and targeted the `/proc` filesystem directly using the identified Process ID (PID 41).
    
    ```Bash
    ls /proc/41
    ```
    
4.  **Memory Extraction:** I first checked the raw command line execution state. I then dumped the environment variables loaded directly into the process memory. This bypassed any disk-level wiping and revealed the hidden `ANALYST_KEY` variable containing the flag.

    ```Bash
    cat /proc/41/cmdline
    cat /proc/41/environ
    ```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1160251.svg)](https://asciinema.org/a/1160251)

**Command Breakdown**
|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`ps`|`aux`|Displays all running processes on the system, including those owned by other users, with detailed resource usage.|
|`ps`|`-q`|Filters the process list to only display the specified Process ID (PID).|
`cat`|None|Concatenates and prints the raw contents of a file or memory state directly to standard output.|
|`ls`|None|Lists the state files and subdirectories available within a specific `/proc` PID folder.|

## **Notes & Takeaways**

Wiping disk logs is standard attacker behavior. To find the truth during an active incident, you must look at what is actively loaded in RAM. Standard tools like `ps` simply parse the `/proc` directory for human readability and can be spoofed. Interrogating `/proc/<PID>/environ` directly reveals the unparsed truth, exposing how attackers pass secrets and configurations to fileless payloads in memory.

### **Understanding Process Memory and the Proc Filesystem**

When you open a terminal and look around a standard Linux system, you are looking at files stored on a hard drive. The `/proc` directory is different. It is an illusion. It is not actually stored on the hard drive at all. It is a "pseudo-filesystem."

Think of `/proc` as a direct, live window into the brain of the Linux kernel. Every time a program runs, the kernel assigns it a Process ID (PID) and creates a temporary folder inside `/proc` named with that number. Inside that folder, the kernel keeps track of absolutely everything that program is doing in live memory (RAM).

When attackers want to hide, they delete their files from the hard drive. They change the names of their programs so that tools like `ps` show a fake name. But they cannot lie to the kernel.

If you look inside `/proc/<PID>/`, you bypass all the fake names and look directly at the memory state.

-   **cmdline:** This file shows the exact, unedited command used to launch the program.
    
-   **environ:** This file holds all the environment variables given to the program when it started. Attackers frequently use environment variables to pass passwords, API keys, or instructions into their malware so they do not have to save those secrets on the hard drive.
    

By running `cat /proc/41/environ`, you completely bypassed the attacker's attempts to hide and extracted the secret password directly out of the live system memory.
