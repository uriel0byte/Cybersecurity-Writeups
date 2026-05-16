# Author: LT 'syreal' Jones

# Description
Download this disk image and find the flag.

Note: if you are using the webshell, download and extract the disk image into /tmp not your home directory.

    Download compressed disk image

# Hints
1.  None

# Disk Analysis
One of the most fundamental skills of a forensics analyst is inspecting and deeply understanding disks. These can be actual hardware or dumps of disks captured in files. There are a few really good GUI tools out there for not just disk analysis, but whole management of digital evidence for cases. Our disk analysis problems will not require any licenses to proprietary software. Some people like to use Autopsy which is a GUI frontend to the tools we will demonstrate how to use in this section. We will use the individual Sleuthkit tools so that you learn a little more than from a GUI that abstracts away some of the details. Disks are all about the details.

More: https://primer.picoctf.org/#_disk_analysis <---- Very helpful.

# Steps
1. Download the disk image `disk.flag.img.gz`.
2. Unzip the compressed data with `gunzip disk.flag.img.gz`. Got the `disk.flag.img` which is a `DOS/MBR boot sector; partition 1`.
3. Used `mmls disk.flag.img` to see the partition tables. To find the offset to the main partition, notice the fourth partition that it has the largest length. 

```bash
urielbyte-academy@webshell:/tmp$ mmls disk.flag.img 
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000206847   0000204800   Linux (0x83)
003:  000:001   0000206848   0000360447   0000153600   Linux Swap / Solaris x86 (0x82)
004:  000:002   0000360448   0000614399   0000253952   Linux (0x83)
```

4.  Copy the 'Start' value and used `fls -o 360448 disk.flag.img ` -o <imageoffset> flag - to specify the offset into image file (in sectors).

```bash
urielbyte-academy@webshell:/tmp$ fls -o 360448 disk.flag.img 
d/d 451:        home
d/d 11: lost+found
d/d 12: boot
d/d 1985:       etc
d/d 1986:       proc
d/d 1987:       dev
d/d 1988:       tmp
d/d 1989:       lib
d/d 1990:       var
d/d 3969:       usr
d/d 3970:       bin
d/d 1991:       sbin
d/d 1992:       media
d/d 1993:       mnt
d/d 1994:       opt
d/d 1995:       root
d/d 1996:       run
d/d 1997:       srv
d/d 1998:       sys
d/d 2358:       swap
V/V 31745:      $OrphanFiles
```

5. Append the inode number to the fls command to list all the files in that diretory. First I checked the home directory with the inode number of 451 but foudn nothing. Then move on to the root directory, abd found my_folder directory which is very suspicious.

```bash
urielbyte-academy@webshell:/tmp$ fls -o 360448 disk.flag.img 451
urielbyte-academy@webshell:/tmp$ fls -o 360448 disk.flag.img 1995
r/r 2363:       .ash_history
d/d 3981:       my_folder
urielbyte-academy@webshell:/tmp$ fls -o 360448 disk.flag.img 3981
r/r * 2082(realloc):    flag.txt
r/r 2371:       flag.uni.txt
```

6. Used `icat` command to print out the file!

```bash
urielbyte-academy@webshell:/tmp$ icat -o 360448 disk.flag.img 2371
picoCTF{by73_5urf3r_adac6cb4}
```

Answer: picoCTF{by73_5urf3r_adac6cb4}
