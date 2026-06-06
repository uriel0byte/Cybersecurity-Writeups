# BreachLab: Ghost - Level 10 → 11

## **Objective**

Identify a single unique string within a dataset containing hundreds of duplicated entries using command-line text processing utilities.

## **Why this matters in the real world**

Log deduplication at scale. The first tool in the belt of every SIEM engineer who has to find the one anomalous event out of a million identical ones.

## **Investigation & Solution**

1.  **Initial Enumeration & Baselining:** Started by listing directory contents to find `data.txt`. Verified it was standard ASCII text using the `file` command, then ran `wc data.txt` to gauge the size of the dataset. The output showed exactly 801 lines, meaning there were 400 duplicate pairs and a single outlier.
    
    ```bash
    ls -la
    file data.txt
    wc data.txt
    ```
    
2.  **Structural Analysis:** Used `head -10` to sample the top of the file. The output confirmed a list of randomized alphanumeric strings with apparent duplicates scattered throughout the dataset.
    
    ```bash
    head -10 data.txt
    ```
    
3.  **Data Processing & Extraction:** To isolate the single anomalous string, I first piped the file contents through `sort` to group all identical strings together. I then piped that sorted output into `uniq -u`, which drops all duplicated entries and outputs only the line that occurs exactly once.

    ```bash
    sort data.txt | uniq -u
    exit
    ```
    

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1164161.svg)](https://asciinema.org/a/1164161)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`file`|None|Determines file type based on headers and magic numbers rather than relying on file extensions.|
|`wc`|None|Prints newline, word, and byte counts for a file (Word Count).|
|`head`|`-10`|Outputs the first 10 lines of a file to quickly sample its structure.|
|`sort`|None|Sorts lines of text files alphanumerically.|
|`uniq`|`-u`|Filters standard input to only print lines that are strictly unique (no duplicates).|

## **Notes & Takeaways**

The most common mistake people make when attempting deduplication on the command line is running `uniq` on its own.

The `uniq` command does not read the entire file into memory; it only compares _adjacent_ lines. If identical strings are separated by other data, `uniq` will not recognize them as duplicates. Therefore, piping data through `sort` is a mandatory prerequisite to group identical lines together before `uniq` can properly filter or count them.
