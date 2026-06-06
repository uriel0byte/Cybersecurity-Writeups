# BreachLab: Ghost - Level 4 → Level 5

## **Objective**

Identify the anomalous log entry within a large dataset of files to retrieve the hidden credentials.

## **Why this matters in the real world**

Threat hunting. This is the core loop of every SOC analyst on the planet — find the needle in the log haystack.

## **Investigation & Solution**

1.  **Enumeration & Pattern Recognition:** Navigated into the `vault` directory, which contained hundreds of `record_` files and an `info.txt` file. Inspected a few files to establish a baseline. The standard format was a 63-byte file containing a `STATUS: <hash>` entry.

    ```bash
    cd vault/
    ls
    ls -l info.txt record_0001 record_0002 record_0431
    head -10 info.txt
    cat record_0001
    ```

2.  **Initial Noise Filtering:** Read `info.txt` and used `grep -v` to strip away the baseline `STATUS` lines. This revealed several decoy `password=` lines and the actual flag, confirming the anomaly was buried in the dataset.

    ```bash
    cat info.txt | grep -v "STATUS"
    ```

3.  **Hunting the Anomaly:** Instead of just pulling from the summary file, I wanted to isolate the specific anomalous records. Since standard records were 63 bytes, I used `find` to locate any files strictly smaller than the baseline.

    ```bash
    find . -type f -size -63c
    ```

4.  **Extraction:** Piped the contents of the anomalous, smaller files directly into `grep` and filtered out the known `password=` decoys to extract the true credential.

    ```bash
    find . -type f -size -63c -exec cat {} + | grep -v "password"
    exit
    ```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1129658.svg)](https://asciinema.org/a/1129658)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`cd`|None|Changes the current directory.|
|`ls`|`-l`|Lists directory contents in a long format to reveal permissions, ownership, file size, and modification dates.|
|`head`|`-10`|Outputs the first 10 lines of a file to quickly sample its structure.|
|`cat`|None|Concatenates and prints the raw contents of a file directly to standard output.|
|`grep`|`-v`|Inverts the match, filtering OUT lines containing the specified string.|
|`find`|`-type f`|Restricts the search to only files.|
|`find`|`-size -63c`|Filters the search for files strictly smaller than 63 bytes (the `c` stands for bytes).|
|`find`|`-exec cat {} +`|Executes the `cat` command efficiently across all files found in a single pass.|
|`exit`|None|Closes the current shell session.|

## **Notes & Takeaways**

This level demonstrates how to handle log exhaustion. When faced with massive amounts of data, you don't look for the bad; you filter out the known good. Using `grep -v` to strip away the baseline `STATUS` noise immediately surfaced the anomalies. Combining that with `find` to identify deviations in expected file size (`-size -63c`) is a highly practical technique for isolating artifacts quickly.
