# BreachLab: Ghost - Level 22 (Graduation)

## **Objective**

Synthesize multiple offensive and forensic techniques—binary string extraction, data decoding, and SUID privilege escalation—to recover three fragmented authentication shards and submit them to a secure local gateway.

## **Why this matters in the real world**

Ghost was selection. This is graduation. You will not solve this with one tool — only by combining everything you practiced. Clear this and you are no longer a beginner, you are an operative. Every future BreachLab track is open to you. The real work starts now.

## **Investigation & Solution**

The objective required locating and extracting three specific shards scattered across the system using different methodologies.

### **Phase 1: Shard 1 (Binary Extraction)**

Inspecting the home directory revealed a binary blob named `relic.bin`. Opening it with a standard text reader would flood the terminal with unreadable characters. I utilized `strings` to extract only the human-readable text from the binary and piped the output into `grep` to isolate the specific shard identifier.

```bash
file relic.bin
strings relic.bin | grep 'SHARD'
```

### **Phase 2: Shard 2 (Decoding)**

The second file, `scroll.b64`, was identified as standard ASCII text. Reading the file revealed a Base64 encoded string. I utilized the `base64` utility with the decode flag (`-d`) to parse the payload back into plaintext.

```bash
cat scroll.b64
base64 -d scroll.b64
```

### **Phase 3: Shard 3 (SUID Privilege Escalation)**

The final shard was guarded by a SUID helper. I executed a system-wide search using `find` to locate any executables owned by `root` but accessible to the `ghost22` group. This revealed the `/usr/local/bin/ghost-archivist` binary. Executing this SUID binary temporarily elevated my privileges, allowing the program to retrieve and print the final shard.

```bash
find /usr -group ghost22 -exec ls -la {} +
cd /usr/local/bin
./ghost-archivist
```

### **Phase 4: Assembly & Submission**

After testing the local gatekeeper service on port 31339 to confirm the expected syntax, I constructed a single pipeline using `echo` and `nc` to submit all three shards simultaneously in the exact required format.

```bash
echo "SHARD1:<>|SHARD2:<>|SHARD3:<>" | nc localhost 31339
exit
```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1200387.svg)](https://asciinema.org/a/1200387)

## **Command Breakdown**

| **Command** | **Flag/Option** | **Purpose** |
| --- | --- | --- |
| `strings` | None | Extracts printable character sequences from binary files. |
| `grep` | `'SHARD'` | Filters standard input to output only lines matching the given pattern. |
| `base64` | `-d` | Decodes a Base64 encoded string back to plaintext. |
| `find` | `-group` / `-exec` | Searches the filesystem for files matching specific criteria (group ownership) and executes a command (`ls -la`) on the results. |

## **Notes & Takeaways**

The culmination of an investigation rarely relies on a single exploit or a single tool. This graduation exercise accurately reflects real-world incident response and penetration testing, where an analyst must seamlessly transition between artifact analysis (binary strings, encoding formats), system enumeration (hunting for SUID misconfigurations), and raw network interaction (socket communication). Documenting these chained techniques clearly in a technical write-up is critical for establishing a verifiable record of exactly how a system was navigated and compromised.
