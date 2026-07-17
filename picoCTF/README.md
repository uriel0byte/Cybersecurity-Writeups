# picoCTF — Challenge Writeups

CTF challenge writeups from picoCTF. Primary focus is **Forensics** and **General Skills** — the two categories most directly applicable to Blue Team and SOC work. Cryptography and Web Exploitation writeups are included as supplementary material.

Each writeup documents the approach, tools used, and key concepts. Challenges where I got stuck or learned something new are noted explicitly rather than glossed over.

---

## Forensics

The core of this section. File analysis, steganography, disk forensics, PCAP analysis, metadata extraction, and file carving — all skills that transfer directly to DFIR and SOC work.

### Easy

| Challenge | Concept | Tools | Status |
|---|---|---|---|
| CanYouSee | Metadata / Base64 | exiftool, CyberChef | ✅ |
| Corrupted File | File header repair (JPEG) | Hex editor | ✅ |
| DISKO 1 | Disk image / string extraction | strings, grep | ✅ |
| Flag in Flame | Base64 decode / image extraction | base64, imagetotext | ✅ |
| Glory of the Garden | Hex dump / string search | xxd, strings, grep | ✅ |
| Hidden in Plainsight | Metadata / Steghide | exiftool, steghide | ✅ |
| Ph4nt0m 1ntrud3r | PCAP / Base64 fragments | Wireshark | ✅ |
| RED | LSB Steganography (PNG) | zsteg, CyberChef | ✅ |
| Riddle Registry | PDF Metadata / Base64 | exiftool, CyberChef | ✅ |
| Scan Surprise | QR Code analysis | Mobile scanner / zbar | ✅ |
| Secret of the Polyglot | Polyglot file (PNG+PDF) | file, binwalk, exiftool | ✅ |
| Verify | SHA-256 checksum verification | sha256sum, grep | ✅ |
| information | Image metadata / Base64 | exiftool, CyberChef | ✅ |

### Medium

| Challenge | Concept | Tools | Status |
|---|---|---|---|
| Disk, disk, sleuth! | Disk image / string search | srch_strings, grep | ✅ |
| Enhance! | SVG XML inspection | browser inspector, cat | ✅ |
| Extensions | File type vs. extension mismatch | file, xxd | ✅ |
| File Types | Nested archive extraction | file, binwalk, lzip, lz4, lzma | ✅ |
| Lookey here | String search in large file | grep | ✅ |
| Matryoshka Doll | Nested image extraction | binwalk | ✅ |
| Redaction Gone Wrong | Improper PDF redaction | PDF reader | ✅ |
| Sleuthkit Apprentice | Disk partition + file recovery | mmls, fls, icat | ✅ |
| Sleuthkit Intro | Partition size analysis | mmls | ✅ |
| SoMeta | XMP metadata analysis | exiftool | ✅ |
| St3g0 | LSB Steganography (PNG) | zsteg | ✅ |
| Trivial Flag Transfer Protocol | TFTP + steganography | Wireshark, steghide, ROT13 | ✅ |
| What Lies Within | LSB Steganography (PNG) | zsteg | ✅ |
| c0rrupt | PNG header repair | hex editor | ✅ |
| hideme | Embedded ZIP in PNG | binwalk, xxd | ✅ |
| m00nwalk | SSTV audio decoding | SSTV decoder | ✅ |
| tunn3l v1s10n | BMP file analysis | file, exiftool, hex editor | 🔄 In Progress |

---

## General Skills

Linux CLI, file operations, encoding/decoding, Git, scripting, and basic network tools. These are the daily-driver skills for any SOC or DFIR role — log parsing, piping output, identifying file types, and working a remote shell efficiently.

### Easy

| Challenge | Concept | Tools | Status |
|---|---|---|---|
| 2warm | Base conversion (decimal → binary) | CyberChef | ✅ |
| Bases | Base64 decoding | CyberChef | ✅ |
| BigZip | Recursive grep in directory | grep -ri | ✅ |
| Binary Search | Algorithm / O(log n) concept | manual | ✅ |
| Blame Game | Git log per file | git log | ✅ |
| Codebook | XOR decryption via Python | python3 | ✅ |
| Collaborative Development | Git branches / merge | git branch, merge | ✅ |
| Commitment Issues | Git history / checkout | git log, git show | ✅ |
| FANTASY CTF | Terminal navigation | nc | ✅ |
| FirstFind | find command | find -exec | ✅ |
| FirstGrep | grep fundamentals | grep | ✅ |
| GlitchCat | Hex to ASCII / Python chr() | CyberChef / Python | ✅ |
| HashingJobApp | Hash identification and generation | online hash tool | ✅ |
| Let's Warm Up | Hex to ASCII | CyberChef | ✅ |
| Log Hunt | grep + sort + uniq pipeline | grep, sort, uniq | ✅ |
| MY GIT | Git impersonation / push | git config, push | ✅ |
| Magikarp Ground Mission | Directory navigation | cd, cat | ✅ |
| Nice netcat | Netcat + ASCII decoding | nc, dCode | ✅ |
| Obedient Cat | cat | cat | ✅ |
| PWCrack1 | Source code analysis | python3 | ✅ |
| PWCrack2 | Hex to ASCII / password extraction | python3 | ✅ |
| ping-cmd | OS Command Injection | nc | ✅ |
| Printer Shares | SMB enumeration | smbclient | ✅ |
| Static ain't always noise | strings on binary | strings, grep | ✅ |
| Super SSH | SSH connection | ssh | ✅ |
| Tab, Tab, Attack | Tab-complete / directory traversal | bash | ✅ |
| Time Machine | Git commit messages | git log | ✅ |
| Undo | Linux text transformation reversal | base64, rev, tr | ✅ |
| WarmedUp | Hex to decimal | CyberChef | ✅ |
| Wave a flag | Help flags on binaries | chmod, ./binary -h | ✅ |
| binhexa | Binary / hex operations | online calculator | ✅ |
| convertmepy | Decimal to binary | Python3 | ✅ |
| repetitions | Nested Base64 | CyberChef | ✅ |
| runme.py | Running Python scripts | python3 | ✅ |
| strings it | strings on ELF binary | strings, grep | ✅ |
| what's a netcat | Netcat basics | nc | ✅ |
| Multicode | Nested encoding | CyberChef | ✅ |

### Medium

| Challenge | Concept | Tools | Status |
|---|---|---|---|
| ASCII Numbers | Hex to ASCII | CyberChef | ✅ |
| Nothing Up My Sleeve | wget + cat | wget, cat | ✅ |
| PWCrack3 | Hash cracking + Python automation | python3 | ✅ |
| Permissions | sudo -l / vim privilege escalation | sudo, vim | ✅ |
| Python Wrangling | Python script flags (-e/-d) | python3 | ✅ |
| Serpentine | Source code modification | python3, vim | ✅ |
| plumbing | nc output piping to file + grep | nc, grep | ✅ |
| useless | man pages | man | ✅ |
| Based | Multi-base encoding | — | 🔄 In Progress |

---

## Supplementary Categories

Completed writeups exist for **Cryptography** (ROT13, Caesar, Vigenère, Base64, A1Z26, substitution ciphers, hash cracking) and **Web Exploitation** (HTML inspection, robots.txt, client-side auth bypass, file includes). These are not the primary focus but are included for completeness.

- [Cryptography →](Cryptography/)
- [Web Exploitation →](Web%20Exploitation/)

---

## Tools Reference

| Tool | Used For |
|---|---|
| exiftool | Metadata extraction from images and PDFs |
| binwalk | Embedded file detection and extraction |
| zsteg | LSB steganography in PNG/BMP |
| steghide | Passphrase-protected steganography in JPEG/WAV |
| strings / srch_strings | Printable string extraction from binaries and disk images |
| xxd | Hex dump and file header inspection |
| mmls / fls / icat | Disk partition and file system analysis (Sleuth Kit) |
| Wireshark | PCAP analysis and protocol filtering |
| CyberChef | Encoding/decoding Swiss Army knife |
| grep / sort / uniq | Log parsing and data extraction pipelines |
| git log / show / checkout | Git history forensics |
| file / wc / head | File type identification and preliminary triage |

---

*Supawat H. (uriel0byte) — Blue Team / SOC Tier 1 Practice*
