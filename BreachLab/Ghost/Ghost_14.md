# BreachLab: Ghost - Level 14 → 15

## **Objective**

Establish a TLS-encrypted connection to a local service and transmit the current credentials to retrieve the next flag.

## **Why this matters in the real world**

Ninety-nine percent of traffic on the internet is now TLS. Every real-world recon, every API poke, every banner grab demands a TLS-capable client.

## **Investigation & Solution**

1.  **Reconnaissance Review:** Initial Nmap scans without the `-p-` flag only check the top 1,000 ports, which surfaced the plaintext service on port 30000 but missed higher-range custom listeners. Reviewing the exhaustive 65,535-port `nc` sweep from Level 5 revealed the true target: port `30001`.
    
2.  **Encrypted Transmission:** Standard Netcat cannot negotiate a TLS handshake. To communicate with the encrypted service, I utilized `openssl s_client`. I piped the current password directly into the OpenSSL connection and appended the `-quiet` flag to suppress the verbose certificate chain output, allowing for a clean interaction with the application layer.

    ```bash
    echo "N3tc4t_D3l1v3r" | openssl s_client -connect localhost:30001 -quiet
    ```
    
> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1172437.svg)](https://asciinema.org/a/1172437)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`openssl s_client`|`-connect`|Implements a generic SSL/TLS client which connects to a remote host and port.|
|`openssl s_client`|`-quiet`|Suppresses the printing of the session and certificate information, acting more like a raw encrypted Netcat session.|

## **Notes & Takeaways**

Default tooling assumptions are a massive blind spot. Nmap's default top-1,000 port scan is efficient but incomplete. When performing exhaustive host enumeration, `nmap -p-` or a scripted `nc` loop across all 65,535 ports is mandatory to avoid missing custom malware listeners or non-standard administrative services.

Furthermore, `openssl s_client` is an indispensable tool for the Blue Team. When a service requires TLS, standard `nc` or `telnet` will fail with parsing errors. OpenSSL bridges that gap, and combining it with the `-quiet` flag strips away the cryptographic noise so you can focus strictly on the application payload.
