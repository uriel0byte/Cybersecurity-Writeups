# Author: Yahaya Meddy

# Description
After logging in, you will find multiple file parts in your home directory. These parts need to be combined and extracted to reveal the flag.

# Hints
1. None

# Steps
1. Clicked "Launch Intstance" and connect to the provided server via ssh.
2. Use ls to see all the files.

```
ctf-player@pico-chall$ ls -l
total 24
-rw-r--r-- 1 ctf-player ctf-player 282 Feb  4 21:22 instructions.txt
-rw-r--r-- 1 ctf-player ctf-player  51 Feb  4 22:40 part_aa
-rw-r--r-- 1 ctf-player ctf-player  51 Feb  4 22:40 part_ab
-rw-r--r-- 1 ctf-player ctf-player  51 Feb  4 22:40 part_ac
-rw-r--r-- 1 ctf-player ctf-player  51 Feb  4 22:40 part_ad
-rw-r--r-- 1 ctf-player ctf-player  35 Feb  4 22:40 part_ae
```

3. cat the instruction.txt to have a better understanding of how to process this.
   
```bash
Hint:
- The flag is split into multiple parts as a zipped file.
- Use Linux commands to combine the parts into one file.
- The zip file is password protected. Use this "supersecret" password to extract the zip file.
- After unzipping, check the extracted text file for the flag.
```

4. I tried to unzip one part at a time but it caused an error due to missing central directory which located at the end of the zip file (I guess)

```bash
ctf-player@pico-chall$ unzip part_aa
Archive:  part_aa
  End-of-central-directory signature not found.  Either this file is not
  a zipfile, or it constitutes one disk of a multi-part archive.  In the
  latter case the central directory and zipfile comment will be found on
  the last disk(s) of this archive.
unzip:  cannot find zipfile directory in one of part_aa or
        part_aa.zip, and cannot find part_aa.ZIP, period.
```

5. At first I thought the error was about its missing extension so I used file command to verify it.
6. Use "cat part_* > combined.zip" to cocatenate the raw binary into one zip file.
7. Then unzip it using the provided password from the instruction.txt and got the flag.txt.
8. Use cat to reveal the file's content.

Answer: picoCTF{z1p_and_spl1t_f1l3s_4r3_fun_da494d2e}
