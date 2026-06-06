# BreachLab: Ghost - Level 3 → Level 4

## **Objective**

Understand and navigate file permissions to access restricted files based on group membership.

## **Why this matters in the real world**

Linux permissions are the entire foundation of privilege escalation. This is level zero of real privesc.

## **Investigation & Solution**

1.  **Enumeration:** Started by reading `map.txt` in the home directory. The file contained a storage layout mapping out `/var/intel/public/`, `/var/intel/ops/`, and `/var/intel/archive/`, along with a hint that access follows a group scheme and to ask the kernel for identity.
    
    ```bash
    pwd
    ls -l
    cat map.txt
    ```

2.  **Identity Verification:** Used `id` to determine current user privileges. Confirmed membership in the `analysts` group (`gid=1010`).
    
    ```bash
    id
    ```

3.  **Directory Traversal & Analysis:** * Checked the `public` directory first. Read `report_q1.txt`, which was a decoy pointing toward restricted files.
    * Moved to the `ops` directory. A standard `ls` revealed two files. Read `operative_list.txt`, which directed attention to `access_codes.dat` for credentials.
    
    ```bash
    ls /var/intel/public/
    cat /var/intel/public/report_q1.txt
    ls /var/intel/ops/
    cat /var/intel/ops/operative_list.txt
    ```

4.  **Extraction & Permission Verification:** Extracted the flag from `access_codes.dat`. Attempted to access the `archive` directory but received a "Permission denied" error. Checked the long-format listing of the `ops` directory, confirming that while the files were owned by `root`, they were readable by the `analysts` group. The `archive` directory remained inaccessible, matching the initial `map.txt` layout stating it was "root only."
    
    ```bash
    cat /var/intel/ops/access_codes.dat
    ls /var/intel/archive/
    ls -l /var/intel/ops/
    ls -l /var/intel/archive/
    exit
    ```
    
> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1069116.svg)](https://asciinema.org/a/1069116) 

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`pwd`|None|Prints the absolute path of the current working directory.|
|`ls`|None|Lists standard directory contents.|
|`ls`|`-l`|Lists directory contents in a long format to reveal permissions, ownership, file size, and modification dates.|
|`cat`|None|Concatenates and prints the raw contents of a file directly to standard output.|
|`id`|None|Prints real and effective user and group IDs.|
|`exit`|None|Closes the current shell session.|

## **Notes & Takeaways**

This level highlights basic Linux access control. The `id` command is critical for mapping out what a compromised account can actually do on a system. Understanding the read/write/execute (`rwx`) triplet for User, Group, and Others is essential for spotting misconfigurations that lead to lateral movement or privilege escalation.
