# BreachLab: Ghost - Level 11 → 12

## **Objective**

Identify and systematically reverse multiple layers of file compression (`tar`, `bzip2`, `gzip`) to extract the core payload.

## **Why this matters in the real world**

Real malware payloads are nested three or four layers deep to defeat simple sandboxes. This is the exact loop an analyst runs on a fresh sample — strip each layer, identify the next, keep going.

## **Investigation & Solution**

1.  **Initial Triage (Layer 1):** Started by verifying the file type of `data.wrapped`. The `file` command identified it as a POSIX tar archive. I extracted the archive using `tar`, which yielded a new artifact named `core.txt.gz.bz2`.
    
    ```bash
    ls -la
    file data.wrapped
    tar -xvf data.wrapped
    ```
    
2.  **Decompression (Layer 2):** Checked the new artifact with `file`, identifying it as `bzip2` compressed data. After noting that the `bunzip` alias was not present on the system, I referenced the `bzip2` help documentation and used the `-d` flag to force decompression. This dropped the next layer: `core.txt.gz`.

    ```bash
    file core.txt.gz.bz2
    bzip2 -d core.txt.gz.bz2
    ```
    
3.  **Decompression (Layer 3):** Ran `file` on the remaining artifact, confirming it was `gzip` compressed data. Used `gunzip` to strip the final layer, resulting in `core.txt`.

    ```bash
    file core.txt.gz
    gunzip core.txt.gz
    ```
    
4.  **Payload Extraction:** Verified `core.txt` was standard ASCII text and read the contents to capture the flag.

    ```bash
    file core.txt
    cat core.txt
    exit
    ```
    

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1167271.svg)](https://asciinema.org/a/1167271)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`file`|None|Determines file type based on headers and magic numbers rather than relying on file extensions.|
|`tar`|`-xvf`|Extracts (`-x`) files from an archive, verbosely listing files processed (`-v`), using the specified archive file (`-f`).|
|`bzip2`|`-d`|Forces decompression of a `.bz2` file (functionally identical to `bunzip2`).|
|`gunzip`|None|Decompresses files compressed with `gzip`.|

## **Notes & Takeaways**

The most critical habit demonstrated in this level is the disciplined loop of running the `file` command _after every single extraction_.

While the file extension (`.gz.bz2`) hinted at the compression types, malware authors frequently spoof extensions (e.g., naming a zip file `.pdf`) to trick analysts and automated sandboxes into mishandling the payload. Never trust the extension; always verify the magic numbers with `file` before attempting decompression or execution. Furthermore, understanding how to use the core utility flags (like `bzip2 -d`) is essential when operating in stripped-down environments where convenience aliases (like `bunzip2`) might be missing.
