# BreachLab: Ghost - Level 7 → Level 8

## **Objective**

Identify file types and reverse multiple layers of data obfuscation specifically hex dumps and Base64 encoding to retrieve the hidden credentials.

## **Why this matters in the real world**

Malware analysis. Real-world payloads are almost always encoded two or three times deep to evade simple detection.

## **Investigation & Solution**

1.  **Initial Enumeration & Interrogation:** Started by checking the directory contents, which revealed a single file named `transmission.dat`. Instead of immediately reading it, I used the `file` command to determine its nature, identifying it as standard ASCII text.
    
    ```Bash
    ls -la
    file transmission.dat
    ```
    
2.  **Structural Analysis:** Reading the file with `cat` revealed it was not standard plaintext, but rather a formatted hex dump. Looking at the ASCII representation column on the right side of the output (`RDNjMGQzXzByX0QxMw==.`), the `==` padding strongly indicated a secondary layer of Base64 encoding.
    
    ```Bash
    cat transmission.dat
    ```
    
3.  **Deobfuscation Pipeline:** To extract the payload cleanly, I needed to reverse the operations in order. First, I used `xxd -r` to revert the hex dump back into its raw binary/text format. Next, I piped that output directly into `base64 -d` to decode the resulting string, revealing the plaintext credential.
    
    ```Bash
    xxd -r transmission.dat | base64 -d
    exit
    ```
    

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1160135.svg)](https://asciinema.org/a/1160135)

## **Command Breakdown**

**Command Breakdown**
|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`ls`|`-la`|Lists all directory contents, including hidden files, in long format to verify ownership and permissions.|
|`file`|None|Determines file type based on headers and magic numbers rather than relying on file extensions.|
|`cat`|None|Concatenates and prints the raw contents of a file directly to standard output.|
|`xxd`|`-r`|Reverses a hex dump, converting it back into its original binary or text representation.|
|`base64`|`-d`|Decodes Base64 encoded standard input back to plaintext.|

## **Notes & Takeaways**

Never trust a file extension. Using `file` before `cat` is a mandatory habit in incident response to avoid accidentally executing a binary or blowing up your terminal with raw data.

Additionally, be aware of terminal output formatting. The decoded string `D3c0d3_0r_D13` was printed without a trailing newline character, causing the `ghost7@breachlab:~$` prompt to append directly to the end of the credential on the same line. Recognizing where the payload ends and the shell prompt begins is critical for accurate extraction.
