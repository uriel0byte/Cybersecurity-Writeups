# BreachLab: Ghost - Level 5 → Level 6

## **Objective**

Identify active listening services on the local machine and interact with the correct port to retrieve credentials.

## **Why this matters in the real world**

Network reconnaissance and banner grabbing. The opening move in every pentest.

## **Investigation & Solution**

1.  **Reconnaissance (Living off the Land):** With standard scanning tools potentially unavailable, I used Netcat as a makeshift port scanner to sweep all 65,535 ports. Piped standard error to standard output (`2>&1`) and used extended regex to filter out the noise of failed connections.
    
    ```bash
    nc -vz localhost 1-65535 2>&1 | grep -vE 'failed|refused'
    ```
    *Note: This cleanly revealed active listeners on ports 30000, 30001, 30002, 30100, and 30101, among others.*

2.  **Banner Grabbing & Probing:** Began manually connecting to the discovered non-standard ports to inspect their services. The first few ports (30000-30002) returned no immediate response and were dropped.
    
    ```bash
    nc localhost 30000
    nc localhost 30001
    nc localhost 30002
    ```
    
3.  **Following the Trail:** Connecting to port `30100` exposed an informational banner for "GHOST PROTOCOL — CHANNEL A". It provided an authentication token (`GHOST`) and directed operations to a secure channel on port `30101`.
    
    ```bash
    nc localhost 30100
    ```
    
4.  **Extraction:** Connected to the secure channel on port `30101`, submitted the required token syntax (`AUTHENTICATE: GHOST`), and captured the flag.
    
    ```bash
    nc localhost 30101
    AUTHENTICATE: GHOST
    ```
    
> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1134845.svg)](https://asciinema.org/a/1134845)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`nc`|`-v`, `-z`|Verbose mode (prints connection status) and Zero-I/O mode (scans without sending data).|
|`2>&1`|None|Redirects Standard Error (file descriptor 2) to Standard Output (file descriptor 1) so it can be piped.|
|`grep`|`-vE`|Inverts the match (`-v`) and uses Extended Regex (`-E`) to filter out lines containing either "failed" or "refused".|

## **Notes & Takeaways**

This execution highlights a critical "living off the land" technique. Automated scanners like Nmap are standard, but on a stripped-down box, you have to know how to improvise. Using `nc -vz` combined with a `grep -vE` filter turns a basic networking utility into a highly effective, low-noise port scanner. Furthermore, manually interrogating the ports with `nc` is the only way to interact with custom services that don't speak standard protocols.
