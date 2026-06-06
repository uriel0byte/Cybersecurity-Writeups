# BreachLab: Ghost - Level 2 → Level 3

## **Objective**

Locate credentials stored within hidden files and directories across the file system.

## **Why this matters in the real world**

Forensics and malware persistence analysis. Attackers hide their tools in exactly this way. Defenders hunt exactly this way.

## **Investigation & Solution**

1.  **Enumeration:** Started in the home directory and moved into the `investigation` folder. A standard `ls -l` command revealed two standard text files: `report.txt` and `summary.txt`.
    
    ```bash
    pwd
    ls
    cd investigation/
    ls -l
    ```
    
2.  **Following the Trail:** Read both text files. `report.txt` mentioned "Active leads compartmentalized," and `summary.txt` stated that active source files were moved to a "separate location" and contained no credentials. This indicated the need to look for hidden directories.
    
    ```bash
    cat report.txt
    cat summary.txt
    ```
    
3.  **Uncovering Hidden Assets:** Used the `-a` flag with `ls` to show hidden files (those starting with a `.`). This revealed a hidden directory named `.leads`.
    
    ```bash
    ls -la
    ```

4. **Extraction:** Navigated into `.leads`. A standard `ls` showed nothing, confirming the files inside were also hidden. Another `ls -la` exposed three hidden files: `.source_alpha`, `.source_beta`, and `.source_omega`. The first two contained decoy hashes, but `.source_omega` held the valid target string.
    
    ```bash
    cd ./.leads
    ls
    ls -la
    cat ./.source_alpha
    cat ./.source_beta
    cat ./.source_omega
    exit
    ```
    
> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1068883.svg)](https://asciinema.org/a/1068883)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`pwd`|None|Prints the absolute path of the current working directory.|
|`ls`|None|Lists standard directory contents.|
|`ls`|`-l`|Lists directory contents in a long format to reveal permissions, ownership, file size, and modification dates.|
|`ls`|`-la`|Lists all directory contents, including hidden files and directories (those beginning with a dot), in long format.|
|`cd`|None|Changes the current directory.|
|`cat`|None|Concatenates and prints the raw contents of a file directly to standard output.|
|`exit`|None|Closes the current shell session.|

## **Notes & Takeaways**

Attackers rely on hidden files (`.`) to mask persistence mechanisms or stage data, knowing standard ls scans will miss them. Running `ls -la` has to be muscle memory the second you drop into a new directory. Sorting through the decoy hashes in the alpha and beta files was a solid reminder that not every hidden artifact contains the payload—you have to verify the output.
