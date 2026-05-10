# Day 7: Scan-ta Clause - Network Discovery

**Date:** December 7, 2025  
**Time Spent:** 1 hour  
**Difficulty:** ★★☆☆  
**Category:** Network Security / Reconnaissance  
**Room:** https://tryhackme.com/room/networkservices-aoc2025-jnsoqbxgky

---

## Overview

HopSec locked everyone out of `tbfc-devqa01`, defaced the website, and froze the
SOC-mas pipeline. Only the server IP was available. Using Nmap and several service
enumeration tools, I discovered three hidden services, retrieved three key fragments,
regained admin access, and pulled the final flag from a localhost-only MySQL database
that external scans could not see.

Not particularly challenging since I had used Nmap before. Mostly review with a few
new commands.

---

## What I Learned

### Recon Workflow

```
1. Know your target   - IP address
2. Scan open ports    - Nmap (common ports first, then all 65,535)
3. Enumerate services - Investigate what is behind each port
4. Exploit weaknesses - Find entry points
```

### Nmap

**Basic TCP scan (top 1000 ports):**
```bash
nmap MACHINE_IP
```

Output:
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Covers only the top 1000 of 65,535 TCP ports. Fast but incomplete.

---

**Full TCP scan with banner grabbing:**
```bash
nmap -p- --script=banner MACHINE_IP
```

- `-p-` scans all 65,535 TCP ports
- `--script=banner` retrieves service banners

Output:
```
PORT      STATE SERVICE
22/tcp    open  ssh
|_banner: SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.14
80/tcp    open  http
21212/tcp open  trinket-agent
|_banner: 220 (vsFTPd 3.0.5)
25251/tcp open  unknown
|_banner: TBFC maintd v0.2
```

FTP was running on port 21212 instead of the default 21. Services do not always run
on standard ports. Always scan the full range.

---

**UDP scan:**
```bash
nmap -sU MACHINE_IP
```

Output:
```
PORT   STATE SERVICE
53/udp open  domain
```

TCP and UDP are completely separate sets of 65,535 ports each. Scan both or you will
miss services.

---

**Nmap flags reference:**

| Flag | Purpose |
|---|---|
| `-p-` | All 65,535 TCP ports |
| `-sU` | UDP scan |
| `-sV` | Service version detection |
| `-O` | OS detection |
| `-A` | Aggressive (OS + version + scripts + traceroute) |
| `-F` | Fast (top 100 ports) |
| `--script=banner` | Grab service banners |
| `-oN` | Save output to file |
| `-Pn` | Skip host discovery |
| `-T4` | Faster timing (1=slowest, 5=fastest) |

---

### Service Enumeration

**FTP - Port 21212 (anonymous access)**

```bash
ftp MACHINE_IP 21212
Name: anonymous
# No password needed

ftp> ls
ftp> get tbfc_qa_key1 -    # Display file contents
ftp> !                     # Exit
```

Result: Key fragment 1 retrieved.

Anonymous FTP login is a misconfiguration. No credentials required means anyone
can access it.

---

**Netcat - Port 25251 (custom TBFC app)**

```bash
nc -v MACHINE_IP 25251
# Server: TBFC maintd v0.2 - Type HELP for commands
HELP
# Commands: HELP, STATUS, GET KEY, QUIT
GET KEY
# Returns key fragment 2
QUIT      # or Ctrl+C to exit
```

Result: Key fragment 2 retrieved.

Netcat connects to any service regardless of protocol. If you do not know what is
running on a port, use `nc` to probe it.

---

**DNS TXT record - Port 53 UDP**

```bash
dig @MACHINE_IP TXT key3.tbfc.local +short
```

- `@MACHINE_IP` - use this specific DNS server
- `TXT` - request text records
- `+short` - concise output

Result: Key fragment 3 retrieved.

TXT records store configuration data, SPF records, and verification tokens. Worth
querying during recon.

---

**On-host service discovery (after admin access)**

Once inside the admin panel using the three combined keys:

```bash
ss -tunlp
```

- `-t` TCP sockets
- `-u` UDP sockets
- `-n` numeric, no hostname resolution
- `-l` listening only
- `-p` show process info

Output:
```
# External (0.0.0.0) - visible to Nmap:
0.0.0.0:22     SSH
0.0.0.0:80     HTTP
0.0.0.0:53     DNS
0.0.0.0:21212  FTP
0.0.0.0:25251  TBFC app

# Localhost (127.0.0.1) - invisible to Nmap:
127.0.0.1:3306  MySQL
127.0.0.1:8000  Unknown
127.0.0.1:7681  Unknown
```

Services bound to `127.0.0.1` are only accessible from the host itself. External
Nmap scans cannot see them. This is why MySQL never appeared in the initial scan.

---

**MySQL - Port 3306 (localhost)**

```bash
mysql -D tbfcqa01 -e "show tables;"
mysql -D tbfcqa01 -e "select * from flags;"
```

- `-D tbfcqa01` connects to that specific database
- `-e "query"` executes one query and exits

Result: Final flag retrieved from the `flags` table.

MySQL was accepting unauthenticated connections from localhost. Once inside the host,
the database was fully open.

---

### 0.0.0.0 vs 127.0.0.1

```
0.0.0.0  - Accessible from anywhere on the network
           Visible to external Nmap scans
           
127.0.0.1 - Accessible only from the host itself
            Invisible to external Nmap scans
            Only reachable after initial compromise
```

After gaining access to a host, always run `ss -tunlp` to find internal services
that external scanning missed.

---

## Challenges

Mostly familiar ground. New things picked up: `--script=banner` flag, `mysql -e`
one-liner syntax, and `dig` TXT record query format. The chaining of tools was the
most useful part — each discovery led directly to the next step.

---

## Security+ Alignment

**Domain 3.0 - Security Architecture (18%):** Network architecture, service placement,
attack surface reduction, localhost versus external service exposure.

**Domain 4.0 - Security Operations (28%):** Reconnaissance techniques, vulnerability
scanning, network monitoring, threat hunting.

---

## Key Takeaways

**Nmap quick reference:**
```bash
nmap <IP>                          # Top 1000 TCP ports
nmap -p- --script=banner <IP>      # All TCP + banners
nmap -sU <IP>                      # UDP scan
nmap -sV <IP>                      # Version detection
nmap -A <IP>                       # Aggressive
nmap -p 22,80,3306 <IP>            # Specific ports
nmap -oN output.txt <IP>           # Save to file
nmap -T4 <IP>                      # Faster timing
```

**FTP quick reference:**
```bash
ftp <IP>           # Port 21
ftp <IP> 21212     # Custom port
Name: anonymous    # Anonymous login
ls                 # List files
get <file> -       # Display contents
put <file>         # Upload
!                  # Exit shell
```

**Netcat quick reference:**
```bash
nc -v <IP> <port>    # Connect and probe
nc -u <IP> <port>    # UDP mode
```

**dig quick reference:**
```bash
dig @<IP> TXT <domain> +short    # TXT records
dig @<IP> MX <domain>            # Mail records
dig @<IP> A <domain>             # IPv4 records
dig @<IP> AXFR <domain>          # Zone transfer (misconfiguration)
```

**ss quick reference:**
```bash
ss -tunlp                  # All listening ports
ss -tunlp | grep :3306     # Filter by port
```

**MySQL quick reference:**
```bash
mysql -D <db> -e "show tables;"
mysql -D <db> -e "select * from <table>;"
mysql -D <db> -e "show databases;"
```

**Port reference (must know):**

| Port | Protocol | Service |
|---|---|---|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 5432 | TCP | PostgreSQL |
| 8080 | TCP | HTTP alternate |

**Common misconfigurations to hunt:**

| Misconfiguration | Detection | Risk |
|---|---|---|
| Anonymous FTP | Login attempt with "anonymous" | Open file access |
| Unauthenticated DB | MySQL/PostgreSQL connection without credentials | Full data access |
| Service on non-standard port | Full `-p-` scan | Hidden attack surface |
| Service on 0.0.0.0 unnecessarily | `ss -tunlp` on host | Exposed to network |

**Nmap scan signatures in IDS/SIEM:**
```
Basic scan:    Sequential connections to multiple ports, mostly RST responses
Full scan:     65,535 connection attempts from single source
UDP scan:      ICMP "port unreachable" pattern
Banner grab:   Full connect then immediate disconnect after banner
```

**Detection rules worth building:**
- Single IP hitting more than 100 ports within 60 seconds
- Sequential port access from external IP
- Nmap user-agent string in HTTP logs
