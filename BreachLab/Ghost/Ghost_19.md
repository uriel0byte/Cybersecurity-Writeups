# BreachLab: Ghost - Level 19 → 20

## **Objective**

Automate the brute-forcing of a 4-digit PIN against a local TCP service by writing a custom bash script in a restricted environment.

## **Why this matters in the real world**

Every security engineer writes their own tools. You will write more throwaway scripts in a week than you will run someone else's tools. This is the level where that journey starts.

## **Investigation & Solution**

1. **Service Enumeration:** Before writing the payload, I established a baseline connection to the target service on port 30002 using Netcat to confirm the expected input format. The service required the previous level's password followed by a space and a 4-digit PIN.

```bash
nc localhost 30002
```


2. **Script Development (Restricted Environment):** Interactive editors like `vim` or `nano` are frequently absent on hardened targets. To build the brute-force script directly on the command line, I utilized a Here-Document (Heredoc). The script loops through all 10,000 possibilities (`0000` to `9999`) and initiates a fresh Netcat connection for every attempt, appending the server's responses to `result.txt`.

```bash
cat << 'EOF' > test.sh
#!/bin/bash
for i in {0000..9999}; do
echo "SU1D_Fl1p $i" | nc localhost 30002 >> result.txt
done
EOF

chmod +x test.sh
```


3. **Execution & Extraction:** Executed the script and allowed it to finish testing the keyspace. Instead of manually scrolling through thousands of lines of output, I used `grep -v` to invert the match, filtering out every line containing "Wrong PIN". This instantly isolated the single successful server response containing the flag.
```bash
./test.sh
grep -v 'Wrong PIN' result.txt
exit

```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1200376.svg)](https://asciinema.org/a/1200376)

## **Command Breakdown**

| **Command** | **Flag/Option** | **Purpose** |
| --- | --- | --- |
| `cat` | `<< 'EOF'` | Initiates a Here-Document (Heredoc), allowing multi-line text to be redirected into a file without an interactive editor. The single quotes prevent premature variable expansion. |
| `chmod` | `+x` | Modifies file permissions to make the script executable. |
| `grep` | `-v` | Invert match; selects and prints only the lines that do *not* match the specified pattern. |

## **Notes & Takeaways**

**Living off the Land:**
When operating on bastion hosts or containerized environments, interactive text editors are rarely installed. Mastering `cat << 'EOF' > file.sh` is mandatory for writing rapid, multi-line throwaway scripts directly from the standard input prompt.

**The TCP Socket Trap:**
A common mistake when scripting network automation is piping a large text file directly into a single `nc` command (e.g., `cat payload.txt &#124; nc localhost 30002`). Authentication services are designed to drop the TCP connection immediately after a failed attempt. If the first line fails, the socket closes, causing a "Broken Pipe" error for the remaining 9,999 attempts. Effective brute-force automation requires initiating a brand new network connection for every individual payload delivery, which is why the `nc` command must exist *inside* the execution loop.
