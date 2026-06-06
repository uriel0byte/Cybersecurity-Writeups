# BreachLab: Ghost - Level 0 → Level 1

## **Objective**

Navigate the initial file system and read plain text files to locate the first set of target credentials.

## **Why this matters in the real world**

Getting your bearings on a box you have never seen before. Every single engagement — offensive or defensive — starts here.

## **Investigation & Solution**

1.  **Enumeration:** Started by checking my current location and listing out the directory contents. Found a `README` and a `workspace` directory.
    
    ```bash
    pwd
    ls -l
    cat README
    ```
    
   *Note: The README from "KAEL" mentioned leaving in a hurry and explicitly stated that nothing was hidden.*
    
2.  **Following the Trail:** Moved into the `workspace` directory to see what Kael left behind. Found and read `notes.txt`, which explicitly stated that passwords aren't stored in plaintext notes and pointed toward an `archive` directory.
    
    ```bash
    cd workspace/
    ls -l
    cat notes.txt
    ```
    
3.  **Extraction:** Navigated into the `archive` directory, found the `credentials` file, and printed the contents to capture the flag.
    
    ```bash
    cd archive/
    ls
    cat credentials
    exit
    ```
    
> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1068878.svg)](https://asciinema.org/a/1068878)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`pwd`|None|Prints the absolute path of the current working directory to get initial bearings on the host.|
|`ls`|`-l`|Lists directory contents in a long format to reveal permissions, ownership, file size, and modification dates.|
|`cd`|None|Changes the current directory to navigate through the file system.|
|`cat`|None|Concatenates and prints the raw contents of a file directly to standard output.|
|`exit`|None|Closes the current shell or SSH session securely.|

## **Notes & Takeaways**

This was a straightforward warmup for basic terminal navigation (`cd`, `ls`, `cat`), but it introduced a critical rule for BreachLab: **save your flags.** Each level's flag is the SSH password for the next user, and the platform does not log them for you. Just like in real ops, credentials need to be secured locally the exact moment they are obtained. I'll be logging all future flags directly into a password manager as I go.