# BreachLab: Ghost - Level 1 → Level 2

## **Objective**

Read files with special characters and spaces in their filenames to retrieve the next level's credentials.

## **Why this matters in the real world**

Shell quoting is the foundation for shell injection, path traversal, and every real attack that abuses how operators pass arguments to other programs.

## **Investigation & Solution**

1.  **Enumeration:** Started by checking my current location and listing out the directory contents. Found a `MANIFEST` file alongside three unusually named files: `-`, `--help`, and `'file name'`. 
    
    ```bash
    pwd
    ls -l
    cat MANIFEST
    ```
    
   *Note: The MANIFEST from "KAEL" stated that the files were named this way to watch careless analysts give up before reading them.*
    
2.  **Bypassing Shell Interpretation & Extraction:** The core issue is that commands like `cat -` or `cat --help` will cause the shell to interpret the filenames as flags or arguments rather than literal files. To bypass this, I specified the exact relative path (`./`) to force the shell to treat them as filenames and read their contents.
    
    ```bash
    cat ./'file name'
    cat ./-
    cat ./--help
    exit
    ```
    
> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1068881.svg)](https://asciinema.org/a/1068881)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`pwd`|None|Prints the absolute path of the current working directory.|
|`ls`|`-l`|Lists directory contents in a long format to reveal permissions, ownership, file size, and modification dates.|
|`cat`|None|Concatenates and prints the raw contents of a file directly to standard output.|
|`exit`|None|Closes the current shell or SSH session securely.|

## **Notes & Takeaways**

This level essentially combines the concepts from OverTheWire Bandit Levels 1 and 2 into a single challenge. The biggest takeaway here is escaping special characters. When dealing with filenames that start with dashes or contain spaces, using the relative path `./` or wrapping the name in quotes prevents the terminal from getting confused and throwing an error.
