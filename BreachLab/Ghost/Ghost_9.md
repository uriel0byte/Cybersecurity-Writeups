# BreachLab: Ghost - Level 9 → 10

## **Objective**

Extract human-readable strings from a raw binary data file to uncover hidden credentials.

## **Why this matters in the real world**

Malware string analysis. Before you reverse a sample you run strings to find URLs, flags, and the personality of whoever wrote it. This is the single highest-ROI move in early-stage sample triage.

## **Investigation & Solution**

1.  **Enumeration:** Started by listing directory contents, which revealed a hidden `.classified` text file and a `signal.bin` file. Reading the `.classified` file provided flavor text about a "signal" still running at "FREQ: 41.337".

    ```Bash
    ls -la
    cat ./.classified
    ```
    
2.  **File Analysis:** Before interacting with `signal.bin`, I used the `file` command to determine its type. It returned `data`, indicating it was a raw binary file rather than standard text. A quick `ls -lh` confirmed it was a relatively small 13K file.
    
    ```Bash
    file signal.bin
    ls -lh signal.bin
    ```
    
3.  **String Extraction:** To extract readable characters from the binary noise, I used the `strings` command. I piped the output to `nl` to number the lines, allowing me to scroll through and visually identify the flag `== N01s3_Fl00r ==` on line 96.
    
    ```Bash
    strings signal.bin | nl
    exit
    ```
    
> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1164152.svg)](https://asciinema.org/a/1164152)


## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`file`|None|Determines file type based on headers and magic numbers rather than relying on file extensions.|
|`ls`|`-lh`|Lists directory contents in a long, human-readable format (e.g., displaying sizes in K, M, G).|
|`strings`|None|Extracts and prints printable character sequences from a binary file.|
|`nl`|None|Numbers the lines of files or standard input.|

## **Notes & Takeaways**

Never `cat` a binary file directly; it will flood your terminal with unprintable characters and can corrupt your session formatting. Utilizing `strings` is the mandatory first step when triaging an unknown sample to quickly spot hardcoded indicators like URLs, IP addresses, or credentials.

---

### **Advanced Triage: Automated String Hunting**

While manual scrolling via `nl` works for small payloads, real-world malware analysis requires automated filtering to cut through massive amounts of binary noise.

**1. The "Complex String" Regex Method**

Instead of relying on rigid character counts which might miss shorter flags, we can use Extended Regular Expressions (`grep -E`) to hunt for the _structure_ of a typical flag or hardcoded credential. Most flags and generated passwords contain a mix of letters, numbers, and an underscore.

```Bash
# Hunts for any string containing alphanumeric blocks separated by an underscore
strings signal.bin | grep -E "[A-Za-z0-9]+_[A-Za-z0-9]+"

# Alternative: Filters for all strings 8 characters or longer, then searches for an underscore
strings signal.bin | grep -E ".{8,}" | grep "_"
```

**2. The "Debug Padding" Method**

Malware developers frequently leave debug logs, console outputs, and execution markers inside their compiled payloads. To make these logs readable to themselves during development, they use distinct visual padding. Hunting for this padding is one of the fastest ways to locate high-value IOCs (Indicators of Compromise) and hidden payloads.

*Note:* Use `grep -F` (Fixed Strings) or escape special characters like `*` and `[` when searching for these, so the shell does not misinterpret them as regex operators and just look for the literal characters.

-   **Equals Blocks**
    -   _Usage:_ Emphasizing payloads, headers, or flags (e.g., `== N01s3 ==`)
    -   _Command:_ `strings signal.bin | grep -F '=='`
        
-   **Execution Hooks**
    -   _Usage:_ Denoting successful process injection (e.g., `[+] Hook injected`)
    -   _Command:_ `strings signal.bin | grep -F '[+]'`
        
-   **Status Indicators**
    -   _Usage:_ Denoting active tasks or variables (e.g., `[*] Decrypting...`)
    -   _Command:_ `strings signal.bin | grep -F '[*]'`
        
-   **Error/Fail States**
    -   _Usage:_ Denoting failed bypasses or drops (e.g., `[-] Evasion failed`)
    -   _Command:_ `strings signal.bin | grep -F '[-]'`
        
-   **Asterisk Banners**
    -   _Usage:_ Separating major code blocks (e.g., `*** CONFIG ***`)
    -   _Command:_ `strings signal.bin | grep -F '***'`
        
-   **Dash/Hash Lines**
    -   _Usage:_ Visual breaks in long output streams (e.g., `--- END ---`)
    -   _Command:_ `strings signal.bin | grep -E '(---|###)'`
