# Author: syreal

# Description
Use srch_strings from the sleuthkit and some terminal-fu to find a flag in this disk image.

dds1-alpine.flag.img.gz

# Hints
1.  Have you ever used file to determine what a file was?
2.  Relevant terminal-fu in picoGym: https://play.picoctf.org/practice/challenge/85
3.  Mastering this terminal-fu would enable you to find the flag in a single command: https://play.picoctf.org/practice/challenge/48
4.  Using your own computer, you could use qemu to boot from this disk!

# Disk Analysis
One of the most fundamental skills of a forensics analyst is inspecting and deeply understanding disks. These can be actual hardware or dumps of disks captured in files. There are a few really good GUI tools out there for not just disk analysis, but whole management of digital evidence for cases. Our disk analysis problems will not require any licenses to proprietary software. Some people like to use Autopsy which is a GUI frontend to the tools we will demonstrate how to use in this section. We will use the individual Sleuthkit tools so that you learn a little more than from a GUI that abstracts away some of the details. Disks are all about the details.

More: https://primer.picoctf.org/#_disk_analysis <---- Very helpful.

# Steps
1. Download the disk image `dds1-alpine.flag.img.gz`.
2. Unzip the compressed data with `gunzip dds1-alpine.flag.img.gz`. Got the `dds1-alpine.flag.img` which is a `DOS/MBR boot sector; partition 1`.
3. Used `man srch_strings` to get a briefly understand of what does this tool do.
4. Used `srch_strings dds1-alpine.flag.img | grep -i "pico"`. `srch_strings` - display printable strings in files and `grep -i` to narrow the the output to just show lines contain the word "pico" incase-sensitive.

```bash
urielbyte-academy@webshell:/tmp$ srch_strings dds1-alpine.flag.img | grep -i "pico"
ffffffff81399ccf t pirq_pico_get
ffffffff81399cee t pirq_pico_set
ffffffff820adb46 t pico_router_probe
# CONFIG_HID_PICOLCD is not set
  SAY picoCTF{f0r3ns1c4t0r_n30phyt3_5e56e786}
```

5. Research more tools about disk forensics like `mmls` `fls` `icat` `blkcat`, preparing for other disk analysis challenges.

**Note:** Both `srch_strings` and `strings` are command-line utilities used to extract and display printable sequences of characters from binary files, but they serve different user bases and have different feature sets.
If you are just doing basic reverse engineering or checking a compiled binary, `strings` is the tool for the job. If you are doing advanced digital forensics and incident response (DFIR) across disk images, you will likely use `srch_strings`.

Answer: picoCTF{mm15_f7w!}
