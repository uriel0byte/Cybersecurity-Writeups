# Author: Janice He

# Description
Oops! Someone accidentally sent an important file to a network printer—can you retrieve it from the print server?

Additional details will be available after launching your challenge instance.

# Hints
1.  knowing how SMB protocol works would be helpful!
2.  smbclient and smbutil are good tools

# Steps
1. Use given `nc -vz mysterious-sea.picoctf.net 55826` command to check if the printer is online.
2. Use `smbclient -L //mysterious-sea.picoctf.ne -p 55826 -N` command to list all the SMB Shares.
```
Sharename       Type      Comment
        ---------       ----      -------
        shares          Disk      Public Share With Guests
        IPC$            IPC       IPC Service (Samba 4.19.5-Ubuntu)
SMB1 disabled -- no workgroup available
```
3. Use `smbclient //mysterious-sea.picoctf.ne/shares -p 55826 -N` to connect to the share. See the `smb: \>` prompt.
4. Use `dir` command to list all the files on the share.
```
smb: \> dir
  .                                   D        0  Fri Mar  6 20:25:44 2026
  ..                                  D        0  Fri Mar  6 20:25:44 2026
  dummy.txt                           N     1142  Wed Feb  4 21:22:17 2026
  flag.txt                            N       37  Fri Mar  6 20:25:44 2026

                65536 blocks of size 1024. 58696 blocks available
```
5. Use `get flag.txt` command to retrieved the flag file.
6. Use `cat flag.txt` to read it locally.
```
smb: \> get flag.txt 
getting file \flag.txt of size 37 as flag.txt (9.0 KiloBytes/sec) (average 9.0 KiloBytes/sec)

smb: \> exit

urielbyte-picoctf@webshell:~$ cat flag.txt
picoCTF{5mb_pr1nter_5h4re5_51f37693}
```

Answer: picoCTF{5mb_pr1nter_5h4re5_51f37693}
