# BreachLab: Ghost - Level 16 → 17

## **Objective**

Identify the single line of difference between two nearly identical text files to extract the hidden credential.

## **Why this matters in the real world**

Code review, config drift detection, forensic comparison of a known-good baseline to a compromised system. diff is a core skill of every SOC and DFIR engineer.

## **Investigation & Solution**

1. **Enumeration:** Listed the directory contents, revealing two files: `passwords.new` and `passwords.old`. Verified both were standard ASCII text using the `file` command.

```bash
ls -la
file passwords.new
file passwords.old
```


2. **Structural Baselining:** Sampled the first five lines of both files using `head -5` to establish their structure. Both contained identical entries formatted as `entry_XXXX: <hash>`, confirming they were paired datasets.

```bash
head -5 passwords.new
head -5 passwords.old
```


3. **Data Comparison & Extraction:** Executed a direct comparison using `diff`. The tool immediately stripped away all identical lines and isolated the single discrepancy on line 42, revealing the target flag in the newer file.

```bash
diff passwords.old passwords.new
exit
```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1182210.svg)](https://asciinema.org/a/1182210)

## **Command Breakdown**

| **Command** | **Flag/Option** | **Purpose** |
| --- | --- | --- |
| `file` | None | Determines file type based on headers and magic numbers. |
| `head` | `-5` | Outputs the first 5 lines of a file to quickly sample its structure. |
| `diff` | None | Compares two files line by line and outputs the differences. |

## **Notes & Takeaways**

When dealing with massive configuration files or log dumps, reading by eye is guaranteed to result in missed indicators. The `diff` utility is mandatory for detecting unauthorized changes.

It is also crucial to understand how to read raw `diff` output. In this execution, the output started with `42c42`. This is a specific instruction set:

* **42**: Line 42 in the first file (`passwords.old`).
* **c**: Indicates a "(c)hange" occurred (other common indicators are `a` for add and `d` for delete).
* **42**: Line 42 in the second file (`passwords.new`).

The `<` symbol denotes the line from the first file, while the `>` symbol denotes the line from the second file, making it immediately clear exactly what data drifted.
