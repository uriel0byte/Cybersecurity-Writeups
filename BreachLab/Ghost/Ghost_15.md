# BreachLab: Ghost - Level 15 → 16

## **Objective**

Perform an exhaustive port scan to enumerate all active listeners, identify an anomalous TLS-enabled service, and establish an encrypted connection to retrieve credentials.

## **Why this matters in the real world**

Discovery is the first step in every engagement. If you cannot tell the difference between a closed port, a filtered port, and an open port speaking an unexpected protocol, you will miss the entry point.

## **Investigation & Solution**

1. **Exhaustive Reconnaissance:** To establish a complete picture of the host's attack surface, I executed a full 65,535-port Nmap scan. I included service version detection (`-sV`) and double verbosity (`-vv`) to force Nmap to attempt protocol identification on any open listener it found.

```bash
nmap -sV -vv -p- localhost
```


2. **Triage & False Positives:** The scan revealed several non-standard ports. I initially investigated port 30002, but an OpenSSL connection failed. Falling back to a raw plaintext connection via Netcat revealed an informational banner indicating it was a placeholder for a future level (Level 19).

```bash
nc localhost 30002
```


3. **Encrypted Transmission:** Returning to the Nmap output, I identified port 31790 which Nmap flagged as `ssl/unknown`. Recognizing this as the likely target, I used `openssl s_client` to establish a TLS tunnel and piped the current password into the stream, successfully capturing the next flag.

```bash
echo "TLS_0r_N0th1ng" | openssl s_client -connect localhost:31790
```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1182207.svg)](https://asciinema.org/a/1182207)

## **Command Breakdown**

| **Command** | **Flag/Option** | **Purpose** |
| --- | --- | --- |
| `nmap` | `-sV` | Probes open ports to determine service and version information. |
| `nmap` | `-vv` | Double verbosity; provides real-time updates and more detailed scan output. |
| `nmap` | `-p-` | Scans all 65,535 ports rather than just the default top 1,000. |
| `nc` | None | Used here as a raw TCP client to read the plaintext banner of a suspected false-positive port. |
| `openssl s_client` | `-connect` | Implements a generic SSL/TLS client to connect to a remote host and port. |

## **Notes & Takeaways**

Relying exclusively on automated scanner output without manual verification leads to dead ends. While Nmap successfully found all open ports, its service detection heuristics couldn't definitively identify the custom application running on port 31790 (`ssl/unknown`).

Furthermore, running into the "wrong version number" SSL error on port 30002 and immediately pivoting to a raw Netcat connection to read the plaintext banner is a critical troubleshooting reflex. When encrypted connections fail instantly on unknown ports, always test for a plaintext response before moving on.
