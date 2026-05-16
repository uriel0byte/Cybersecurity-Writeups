# Author: LT 'syreal' Jones

# Description
Download the disk image and use mmls on it to find the size of the Linux partition. Connect to the remote checker service to check your answer and get the flag.

Note: if you are using the webshell, download and extract the disk image into /tmp not your home directory.

# Hints
1.  None

# What is Sleuthkit


# Steps
1. Download the disk image `disk.img.gz`.
2. Unzip the compressed data with `gunzip disk.img.gz`. Got the `disk.img` which is a `DOS/MBR boot sector; partition 1`.
3. Used `man mmls` to get a briefly understand of what does this tool do.
4. Used `mmls disk.img` or `mmls -a disk.img` the -a flag only shows the allocated volumns.

```
urielbyte-academy@webshell:/tmp$ mmls disk.img 
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000204799   0000202752   Linux (0x83)
```

5. Connect to a remote server to check and anwser the question.
```
urielbyte-academy@webshell:/tmp$ nc saturn.picoctf.net 55207
What is the size of the Linux partition in the given disk image?
Length in sectors:
```

Answer: picoCTF{mm15_f7w!}
