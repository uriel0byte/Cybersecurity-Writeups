# OverTheWire Bandit Levels 0-33

Published - 2025

**Goal:** Solve all levels of the OverTheWire Bandit wargame to practice fundamental Linux commands.

---

# Level 0 -> 1

**Challenge:** The password for the next level is in a file called `readme` in the home directory.

**Methodology:**
1.  Logged in as `bandit0` using the provided password.
2.  Used `pwd` to print current directory path.
3.  Used the `ls` command to list the files in the current directory with `-l` flag to long listing format.
4.  Found the `readme` file.
5.  Used `file` to detetrmine the file type.
6.  Used the `cat` command to read the contents of the `readme` file.
7.  Copied the password and used it to log in to the next level.

[![asciicast](https://asciinema.org/a/6JgAIb20gnx1t2wWjn27kfEk1.svg)](https://asciinema.org/a/6JgAIb20gnx1t2wWjn27kfEk1)

**Commands Used:**
`pwd`
`ls`
`file`
`cat`
---

# Level 1 -> 2

**Challenge:** The password for the next level is stored in a file called - located in the home directory

**Methodology:**
1.  Logged in as `bandit1` using the password we got from the last time.
2.  Used `pwd` to print current directory path.
3.  Used the `ls` command to list the files in the current directory.
4.  Found the `-` file.
5.  Used `file` to detetrmine the file type.
6.  *The problem with this is our terminal interprets the HYPHEN as flags.*
7.  *Instead of specifying the name of the file, we specify the path to the file, which are Absolute path or Relative path.*
8.  Used the `cat` command to read the contents of the `-` file by using `cat ./-`. or `cat ~/-` or `cat /home/bandit1/-`

[![asciicast](https://asciinema.org/a/YEoDfLj1H4YgcSE3VlmVLVwOV.svg)](https://asciinema.org/a/YEoDfLj1H4YgcSE3VlmVLVwOV)

**Commands Used:**
`pwd`
`ls`
`file`
`cat`
---

# Level 2 -> 3

**Challenge:** The password for the next level is stored in a file called --spaces in this filename-- located in the home directory

**Methodology:**
1.  Logged in as `bandit2` using the password we got from the last time.
2.  Used `pwd` to print current directory path.
3.  Used the `ls` command to list the files in the current directory.
4.  Found the `--spaces in this filename--` file.
5.  *The key problem is that the space character is a delimiter in the Linux shell, used to separate a command from its arguments. If you try to run `cat --spaces in this filename--`, the shell will interpret `--spaces, in, this, and filename--` as four separate arguments, causing an error.*
6. To solve this, you need to escape the spaces. Escaping tells the shell to treat the following character literally, not as a special character. The most common way to do this is with a backslash `\`
7. By running `cat --spaces\ in\ this\ filename--`, you instruct the shell to treat each space as part of the filename. 
8. Another common and often easier method is to enclose the entire filename in quotation marks. This also tells the shell to treat everything inside the quotes as a single argument. The command would be `cat "./--spaces in this filename--"`.
9.  The trick I used in the video down below is `Tab completion` to automatically escape the spaces for you. By typing the first few characters `(--s)` and then pressing the `tab` key twice, the shell will complete the filename and add the necessary backslashes.

[![asciicast](https://asciinema.org/a/tY7ROPUXtUCa6eYCdXMxeBMr7.svg)](https://asciinema.org/a/tY7ROPUXtUCa6eYCdXMxeBMr7)

**Key Takeaway:**
Spaces and other special characters like `*`, `$`, and `&` have a specific meaning to the shell. To treat them as a literal part of a filename or string, you must escape them with a backslash `\` or enclose the entire string in single or double quotes.

**Commands Used:**
`pwd`
`ls`
`cat`
---

# Level 3 -> 4

**Challenge:** The password for the next level is stored in a hidden file in the inhere directory.

**Methodology:**
1.  Logged in as `bandit3` using the password we got from the last time.
2.  Used `pwd` to print current directory path.
3.  Used the `ls -l` command to list the files in the current directory.
4.  Found the `inhere` directory.
5.  Used `cd` to change directory to `inhere` directory then `ls -la`.
6.  *The key problem is that a hidden file has a `.` as the first character of their file name. A normal `ls` or `ls -l` won't show hidden files*
7. To solve this, you need can use `ls` with `-a` flag to show hidden files.
8. Or you can use `find` command to search for all files that start with `.`. By using `find -name ".*"`.
9. `find [directory path] [expression]` in the command above we use `find` without the `[directory path]` leaving it blank will search in current directory.
10. After we found a `Hidden file` we use `cat` command to reveal its content.
   

[![asciicast](https://asciinema.org/a/CPstDvPxH6EQP3OdDcioxfFeV.svg)](https://asciinema.org/a/CPstDvPxH6EQP3OdDcioxfFeV)

**Key Takeaway:**
**Wildcard (*):** Used to represent any sequence of characters, particularly useful when the exact file name is `unknown`. "For times where we do not know the exact file name we can add an aster at the end to search for all files that have names that include what you specified."

**Commands Used:**
`pwd`
`ls -la`
`cat`
`find <path> -name "<>"`
---

# Level 4 -> 5

**Challenge:** The password for the next level is stored in the only human-readable file in the inhere directory.

**Explanation:** `Human-readable` files is files that contain data that is easily read. Like text files and not binary file where software's are requied to understand its content.
There are a types of data of the file (e.g., "`ASCII text`," "`JPEG image data`," "`shell script`," "`ELF`," "`data`"). `ELF` or `data` file, for example, are not human-readable.

The most common data encodings that are human-readable are `ASCII` and `Unicode` such as `UTF-8`.

**Methodology 1:**
1.  Logged in as `bandit4` using the password we got from the last time.
2.  Used `pwd` to print current directory path.
3.  Used the `ls -l` command to list the files in the current directory.
4.  Found the `inhere` directory.
5.  Used `cd` to change directory to `inhere` directory then `ls -la`.
6.  *The key problem is there are a lots of files in this directory. We could just print the contents of every file `cat`. This is, however, not very efficient when we deal with a lot of files. And the Hyphen `(-)` as the first character but we already know how to deal with that*
7. Instead, We use `file ./-*` command to gives us the type of data of the files. We utilize the knowledge we got from before like `./` to specify this current directory and `*` to specify all files. The normal command structure is `file <filename>`
8. This tells us:

`file00`: Looks like text but uses a non-standard encoding

`file07`: A normal ASCII text file ← this one is the most promising

`Others`: Just say "`data`", which usually means it's not readable text (could be binary or garbage)

9. Now we `cat` as usual.


**Methodology 2 (More advanced):**
1.  We can used more advanced commands by combinding `find` `file` (we already know those) and `grep` commands
2.  `grep` command is used to search for patterns or texts within files.
3.  So, the command look like this  ```find -type f -exec file {} + | grep "text"```
  
      💡**Breakdown of the Command**
    
    **`find`**: This is the main command used to search for files and directories within a file system.
    
    **`-type f`**: This option restricts the search to only **files** (not directories, links, or other special file types).
    
    **`-exec file {} +`**: This part executes the `file` command on all the files found by `find`.
    
       **`-exec`**: This is an action that runs a command on each file found.
    
       **`file {}`**: The `file` command determines the **type of a file** (e.g., "ASCII text," "JPEG image data," "shell script"). The `{}` is a placeholder that `find` replaces with the name of each file it finds.
    
       **`+`**: This is a more efficient way to use `-exec`. It bundles the filenames together and passes them to the `file` command in one go, rather than running the command for each file individually.
    
    **`|`**: This is a **pipe**. It takes the output of the command on the left (`find ... -exec file {} +`) and uses it as the input for the command on the right (`grep "text"`).
    
    **`grep "text"`**: This command filters the output from the `file` command, displaying only the lines that contain the string **"text"**. Since the `file` command often describes plain text files as "ASCII text" or "UTF-8 text," this effectively filters for files that are **text-based**.
    
     **In short,** the `find` command locates files of a specific size, and the `grep` command then filters that list to show only the ones that the system identifies as a text file.
    
5.  So the result are `-file00` and `-file07` but the `-file00`: It says `"Non-ISO extended-ASCII text"`, which means it has unusual or outdated character encoding. It might look like gibberish or be unreadable.
6.  Next we do what we always do `cat` it!
    

[![asciicast](https://asciinema.org/a/ZUOHfWa4qJxvmY3MTp5wUcXqB.svg)](https://asciinema.org/a/ZUOHfWa4qJxvmY3MTp5wUcXqB)

**Key Takeaway:**
**Learn more about encoding types**

**Commands Used:**
`ls -la`
`cat`
`file`
`grep`
`find <path> -type <> -exec <> {} +`
---

# Level 5 -> 6

**Challenge:** The password for the next level is stored in a file somewhere under the inhere directory and has all of the following properties:

    human-readable
    
    1033 bytes in size
    
    not executable

**Methodology:**
1.  Logged in as `bandit5` using the password we got from the last time.
2.  Used the `ls -l` command to list the files in the current directory.
3.  Used `cd` to change directory to `inhere` directory then `ls -la`.
4.  Let's try to `ls la` one of the directories to see if it's empty or there are files in it.
5.  *The key problem is there are a lot of directories here. We could `cd` in each directory and print  `cat` the contents of every files then. This is, however, not efficient when we deal with a lot of directories and files.*
6. Now we can utilize the knowledge from the last level, but before we do that let's look at the properties of the file again:

   `human-readble` we already know that normal human-readable files are `ASCII text` or `Unicode` (`UTF-8`).

   `1033 bytes in size` we use flag `-size` and use `1033c` for bytes e.g. suffixes `c` bytes, `k` kilobytes and so on.

   `not executable` we use flag `-executable` and for `not` use `!` exclamation mark before the flag.
   
7. So the command would look like this `find -type f -size 1033c ! -executable -exec file {} + | grep "text"`.
8. Then we `cat` it!

[![asciicast](https://asciinema.org/a/18Z1GZUw2M3PvXD3niG8yFoNN.svg)](https://asciinema.org/a/18Z1GZUw2M3PvXD3niG8yFoNN)

**Key Takeaway:** `!` before a `primary` when using the `find command` to search for the file that `not match` that condition

**Commands Used:**
`cat`
`file`
`grep`
`find <path> -type <> -size <> ! -executable -exec <> {} +`
---

# Level 6 -> 7

**Challenge:** The password for the next level is stored somewhere on the server and has all of the following properties:

    owned by user bandit7
    owned by group bandit6
    33 bytes in size

**Methodology:**
1.  Logged in as `bandit6` using the password we got from the last time.
2.  Used the `ls -la` command to list the files in the current directory.
3.  Found some files but the challenge says that the password is stored somewhere on the server so we have to start searching from root `/`.
4.  Now we can utilize the knowledge from the last level, but before we do that let's look at the properties of the file again. There are new primaries that we do not know:
   
   `owned by user bandit7` we use `-user <name>`
   
   `owned by group bandit6` we use `-group <groups name>`
   
5.  So for this level it does not say that the file is `human-readble` file but let's assume that it's atleast store in a `file`
6.  The command would look like this (do not forget to specify the path, which start from the root `/`):

   `find / -type f -user bandit7 -group bandit6 -size 33c`

7.  *If you try that command, It would show a lot of `Permission Denied` because there are some files that we are not allow to use the command on. Basically, we do not have the enough permissions for those file. You can manaully scroll and find the file that does not show that message, which is pretty hard and not efficient.*
8.  So there is a better way for that, which is that we add `2>/dev/null` after the command.

   `2>/dev/null` hides error messages from your terminal.

   💡Let’s break it down:

   `2` → This refers to "Standard Error" (the channel where error messages go).

   `>` → This means redirect.

   `/dev/null` → This is a special place in Linux that throws things away. It's like a black hole — anything sent here disappears.
   
9.  The final command looks like this: `find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null`
10.  Then we `cat` the file by specifying the Absolute path.

[![asciicast](https://asciinema.org/a/whLGYb4u6g61msqAE6hkpFjkD.svg)](https://asciinema.org/a/whLGYb4u6g61msqAE6hkpFjkD)

**Key Takeaway:** Use `2>/dev/null` to hide unnecessary error messages, so you can focus only on the important output — like when searching through lots of files or running commands that might fail on some inputs.

**Commands Used:**
`ls -la`
`cat`
`find <path> -type <> -size <> -user <name> -group <group>`
---

# Level 7 -> 8

**Challenge:** The password for the next level is stored in the file data.txt next to the word millionth.

**Methodology:**
1.  Logged in as `bandit7` using the password we got from the last time.
2.  Used the `ls` command to list the files in the current directory.
3.  Found the `data.txt`file.
4.  Used `wc` to estimate the content inside. Just for a better understanding of what we are dealing with.

    `wc` shows number of lines, words, and byte count
    
    `-l` shows number of lines
    
    `-w` shows number of words
    
    `-m` shows number of characters
    
6.  We know that `the password` is stored next to the word `millionth`, and we know that `grep` return the entire line of the search pattern.
7.  So we do `cat data.txt | grep "millionth"` or `grep "millionth" data.txt`.

[![asciicast](https://asciinema.org/a/Wpms0rExRwqYurhqs5vpssBXU.svg)](https://asciinema.org/a/Wpms0rExRwqYurhqs5vpssBXU)

**Key Takeaway:** Use `wc` to have a better view of a file. Use `man` to learn a little more of `wc`

**Commands Used:**
`ls`
`cat`
`grep <pattern> <path>`
---

# Level 8 -> 9

**Challenge:** The password for the next level is stored in the file data.txt and is the only line of text that occurs only once

**Methodology:**
1.  Logged in as `bandit8` using the password we got from the last time.
2.  Used the `ls` command to list the files in the current directory. Found the `data.txt`file.
3.  In this clip, I used `file` to see if it's a normal reable file or not and used `wc` to estimate the content inside.
4.  We know the `password` is `the only line of text that occurs only one` so there are tools for that, which are `sort` and `uniq`

    `sort` arranges all the lines in the file alphabetically.
    
    `uniq` compare a line with the **immediately preceding line**. with
           `-u` flag stands for "unique" and tells `uniq` to **only** print lines that appear exactly once in the input.
    
6.  Normal confusion that might happen:

    `uniq data.txt` or `uniq -u` doesn't work as you'd expect because the `uniq` command is designed to work on **adjacent, identical lines**. It requires the input to be sorted first.
    
    `sort data.txt | uniq` this command would give you a list of every single unique line in the file. If a line appeared 50 times, it would be printed once. The single password line would also be printed. This would leave you with a long list to manually search through.
    
    `sort data.txt | uniq -u` this is the right command for this level. we use `sort` first to arranges all the lines in the file alphabetically. This groups all the identical lines together, making them **adjacent** to each other. the `-u` flag is what specifically filters for the **single, non-duplicated** line.
    
7.  Final command : `sort data.txt | uniq -u`
   

[![asciicast](https://asciinema.org/a/DDXus5JYNqki0rklyMl9vglXh.svg)](https://asciinema.org/a/DDXus5JYNqki0rklyMl9vglXh)

**Key Takeaway:** Take it slowly, Slow progress is better than no progress!

**Commands Used:**
`ls`
`cat`
`sort <path>`
`uniq -u <path>`
---

# Level 9 -> 10

**Challenge:** The password for the next level is stored in the file data.txt in one of the few human-readable strings, preceded by several ‘=’ characters.

**Methodology:**
1.  Logged in as `bandit9` using the password we got from the last time.
2.  Used the `ls` command to list the files in the current directory. Found the `data.txt`file.
3.  Used `file` to see if it's a normal reable file or not and used `wc` to estimate the content inside.
4.  We know that the file is `data` type of file which is not human-readable and has a lot of chracters in it. So, Manaully finding it might not be very efficient.
5.  We know that the `password` is stored in `data.txt` in one of the few human-readable strings, preceded by several `=` characters. There is a tool for that `strings`

    `strings <options> <file>` print the sequences of printable characters in files
    
6.  So the command would be `strings data.txt | grep "="`. Do not forget that the password is preceded by serveral `=`.

[![asciicast](https://asciinema.org/a/8dnaOofESCcSPZsp73lxKDXGr.svg)](https://asciinema.org/a/8dnaOofESCcSPZsp73lxKDXGr)

**Key Takeaway:** Use `man` command to learn more about `strings`.

**Commands Used:**
`ls`
`cat`
`grep`
`strings <options> <file>`
---

# Level 10 -> 11

**Challenge:** The password for the next level is stored in the file data.txt, which contains base64 encoded data.

**Methodology:**
1.  Logged in as `bandit10` using the password we got from the last time.
2.  Used the `ls` command to list the files in the current directory. Found the `data.txt`file.
3.  Used `file` to see if it's a normal reable file or not and used `wc` to estimate the content inside.
4.  We know that the file is `ASCII text` type of file which is a human-readable but encoded in base64.
5.  There is a tool for decoding base64 which is `base64`

    `base64 <options> <file>` encode/decode data and print it out.

    `-d` for decoding in `base64`
    
6.  So the command would be `base64 -d data.txt` or `cat data.txt | base64 -d`. 

[![asciicast](https://asciinema.org/a/wepUute14a4fqb1IrbQNMZICL.svg)](https://asciinema.org/a/wepUute14a4fqb1IrbQNMZICL)

**Key Takeaway:** Use `man` command to learn more about `base64`. Learn more about types of encoded data.

**Commands Used:**
`ls`
`cat`
`file`
`wc`
`base64 <options> <file>`
---

# Level 11 -> 12

**Challenge:** The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions

**Methodology:**
1.  Logged in as `bandit11` using the password we got from the last time.
2.  Used the `ls` command to list the files in the current directory. Found the `data.txt`file.
3.  Used `file` to see if it's a normal reable file or not and used `wc` to estimate the content inside.
4.  We know that the file is rotated by 13 position, At first I don't know what to do too, So we have to know about [ROT13](https://en.wikipedia.org/wiki/ROT13) just a bit.
5.  Now we know what Rot13 is, There is a tool for decoding Rot13 which is `tr`

    `tr <options> <string1> <string2>` translate or delete characters.

    
6.  So the command would be `cat data.txt | tr 'A-za-z' 'N-ZA-Mn-za-m`.
   
   The first argument (`'A-Za-z'`) is the set of characters you want to translate. This includes all uppercase letters (A-Z) and all lowercase letters (a-z).

   The second argument (`'N-ZA-Mn-za-m'`) is the replacement set. For a ROT13 cipher, you're shifting each letter by 13 places. The alphabet is 26 letters long, so shifting by      13 splits it in half. The first half (`A-M` and `a-m`) becomes the second half (`N-Z` and `n-z`), and the second half becomes the first. So, `A` becomes `N`, `B` becomes `O`,    and so on, until `M` becomes `Z`. Then the sequence wraps around, so `N` becomes `A`, `O` becomes `B`, and so on.

[![asciicast](https://asciinema.org/a/742087.svg)](https://asciinema.org/a/742087)

**Key Takeaway:** Use `man` command to learn more about `tr`. Learn more about types of encoded data.

**Commands Used:**
`ls`
`cat`
`tr <options> <string1> <string2>`
---

# Level 12 -> 13

**Challenge:** The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work. Use mkdir with a hard to guess directory name. Or better, use the command “mktemp -d”. Then copy the datafile using cp, and rename it using mv (read the manpages!)

**Methodology:**
1.  Logged in as `bandit11` using the password we got from the last time.
2.  Used the `ls` command to list the files in the current directory. Found the `data.txt` file.
3.  Used `file` to see if it's a normal reable file or not and used `wc` to estimate the content inside but we already know that it's hexdump file that has been compressed.
4.  Now let's make a tmp directory at `tmp/<your directory name>`. For me I do `mkdir /tmp/longinus`
5.  Then we `cp data.txt` to that directory `cp data.txt /tmp/longinus/` then we `cd /tmp/logninus`
6.  Before we start decompressing we have to reverse the hexdump file to a binary file first by doing `xxd -r data.txt > data1.bin`
7.  Next we `file` to see what type of compression it is.
8.  We know that it's `gzip` so the command for decompressing is `gunzip` but we have to rename the extension to match the type of file that we found from `file` first. So we do `mv data1.bin data1.gz`
9.  Let's decompress `gunzip data.gzip` then we `ls` and `file` to see what's the next compression type is and `mv` to rename the extension to match it.
10. Then repeated the process. That's is pretty much all it is. In this level we learn about compression types here are the commands and types in this level:

    `gzip` -> `gunzip <file>` or `gzip -d <file>` file extension: `.gz`
    
    `bzip2` -> `bunzip2 <file>` or `bzip2 -d <file>` file extension: `.bz`
    
    `tar` -> `tar -xf <file>` file extension: `.tar`
    
    -x: This flag means extract — it tells tar you want to extract files from an archive.

    -f: This flag means file — it tells tar that the next argument is the name of the archive file you want to work with.
11. The finally file type is `ASCII text` let's just `cat` it

[![asciicast](https://asciinema.org/a/742092.svg)](https://asciinema.org/a/742092)

**Key Takeaway:** Review this level commands.

**Commands Used:**
`ls`
`cat`
`mkdir`
`cp`
`mv`
`file`
`gzip`
`bzip2`
`tar`
---

# Level 13 -> 14

**Challenge:** The password for the next level is stored in /etc/bandit_pass/bandit14 and can only be read by user bandit14. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. Note: localhost is a hostname that refers to the machine you are working on

**Methodology:**
1.  Logged in as `bandit13` using the password we got from the last time.
2.  Used the `ls` command to list the files in the current directory. Found the `sshkey.private` file.
3.  Used `file` to see what type of file it is.
4.  There are 4 ways I can do think of. 1.copy the content manaully 2. use `scp` command 3. use `telnet` command 4. use `nc` command. I don't know much about `telnet` and `nc` so I will only show first two ways.
5.  First, we copy the content inside to a new file in our local machine that has the same name as the original file.
6.  Or Second, we use command `scp` to copy a file from a remote host to our local host so that command look like this `scp -P 2220 bandit13@bandit.labs.overthewire.org:/home/bandit13/sshkey.private ./`

    `scp` stands for secure copy; it works exactly the way the cp command does, but allows you to copy from one host over to another host on the same network. It works via ssh, so all your actions are using the same authentication and security as ssh.

    `scp username@remotehost.com:/remote/directory/myfile.txt /local/directory/`

7.  After i used `scp` command, I have to provide a password of a user again(Bandit13)
8.  After we got the file, we have to change the permission of the file first because in the video I forgot to change the permission of the file and the server respond back with `Permission 0640 for sshkey.private are too open. It is required that private key files are not accessible by others`. So we change permission by using `chmod` command. The command look like this `sudo chmod 700 sshkey.private`
9.  Then we use `ssh` to connect to bandit14 by using `-i <location>` flag to identify the sshkey file so we `ssh -i ./sshkey.private bandit14@bandit.labs.overthewire.org -p2220`
10.  We successfully login into Bandit14. and we can go grab the normal password at `/etc/bandit_pass/bandit14` if you want to.

[![asciicast](https://asciinema.org/a/742265.svg)](https://asciinema.org/a/742265)

**Key Takeaway:** Learn more about Permissions in Linux, `chmod` command, `scp` command.

**Commands Used:**
`ls -l`
`scp`
`chmod`
---

# Level 14 -> 15

**Challenge:** The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

**Methodology:**
1.  Logged in as `bandit14` using the sshkey we got from the last time.
2.  Used the `ls` command to list the files in the current directory. Found nothing then i remember that the last level said that the password is stored at `/etc/bandit_pass/bandit14`.
3.  Used `cd` to or just `cat` the `/etc/bandit_pass/bandit14`.
4.  So I think of `nc` because I've seen it somewhere on the internet so I do some `man nc` and tried lots of command.
5.  After a while i realize how to use `nc` so that command i used is `nc localhost 30000`.
6.  After I do that, I realize that I have to summit the password in order to get the next level password so I `Ctrl + Insert` to paste the password I got from `/etc/bandit_pass/bandit14`
7.  Then we got the password!

[![asciicast](https://asciinema.org/a/742708.svg)](https://asciinema.org/a/742708)

**Key Takeaway:** Learn more about `nc`.

**Commands Used:**
`ls`
`cat`
`nc [destination] [port]`
---

# Level 15 -> 16

**Challenge:** The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL/TLS encryption.

**Methodology:**
1.  Logged in as `bandit15` using the password we got from the last time.
2.  After reading the challange I think of `openssl` because I've seen it somewhere so, I do some `man openssl`, I also try to `nc` like last time but didn't work.
3.  After a while i realize how to use basic `openssl` by adding `s_client` command and tried lots of command. So that final command i used is `openssl s_client localhost:30001`.
4.  After I do that, I have to summit the password in order to get the next level password so I `Ctrl + Insert` to paste the password of Bandit15 that we used to login.
5.  Then we got the password!

[![asciicast](https://asciinema.org/a/742709.svg)](https://asciinema.org/a/742709)

**Key Takeaway:** Learn more about `openssl` command and SSL/TLS encryption just basics like how it works.

**Commands Used:**
`ls`
`openssl`
`openssl s_client [destination:port]`
---

# Level 16 -> 17

**Challenge:** The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

**Methodology:**
1.  Logged in as `bandit16` using the password we got from the last time.
2.  After the last Level, I realized that I can do `man openssl s_client` and I know now that I should specify `-connect` flag and see `CONNECTED COMMAND (BASIC)` but I didn't care at it somehow. That is what make me trying this level for 3hrs.
3.  First I do `man nmap` to see which flags I should use and come across `-p[range]`, `-sV` and `-A` So my command is `nmap -sV -p31000-32000 localhost` You could also do `nmap -A -p31000-32000 localhost` to give more information but it also take more time.
4.  After the results of nmap came out, I notice the `31790` port because it is the most promising one others are `echo` which might mean that it will send back whatever i send. So I do `openssl s_client -connect localhost:30179` and tried to provide the password but didn't work, it keeps saying `KEYUPDATE`. So i did some research on my own at first in man page but no point. So I come across a [Reddit](https://www.reddit.com/r/HowToHack/comments/1g2sivg/bandit_level_16_level_17_keyupdate_problem/) (You should go to this reddit and learn more) and know that I have to specify `-quiet` flag to prevent the The keys k, K, Q, and R from being interpreted. So if the password starts with one of this keystorekes, it will trigger an action. When used with the `-quiet` it won't be the case. So I do `openssl s_client -connect localhost:30179 -quiet` and provide the password and get RSA key back in return.
5.  Copy the key and put it in my local machine by creating a new file `vim sshkey17.private` and paste the key in it.
6.  Change the permission of the file (like we did last time remember?) by doing `sudo chmod 700 sshkey17.private`.
7.  Try to connect to bandit17 `ssh -i sshkey17.private bandit17@bandit.labs.overthewire.org -p2220` and It works.

[![asciicast](https://asciinema.org/a/743064.svg)](https://asciinema.org/a/743064)

**Key Takeaway:** Go read this [Reddit](https://www.reddit.com/r/HowToHack/comments/1g2sivg/bandit_level_16_level_17_keyupdate_problem/) to learn more about `CONNECTED COMMAND` or https://docs.openssl.org/3.0/man1/openssl-s_client/#connected-commands.

**Commands Used:**
`ls`
`nmap`
`nmap -sV -p[range-range] [destination]`
`openssl`
`openssl s_client -connect [destination:port] -quiet`
---

# Level 17 -> 18

**Challenge:** There are 2 files in the homedirectory: passwords.old and passwords.new. The password for the next level is in passwords.new and is the only line that has been changed between passwords.old and passwords.new

NOTE: if you have solved this level and see ‘Byebye!’ when trying to log into bandit18, this is related to the next level, bandit19

**Methodology:**
1.  Logged in as `bandit17` using the private key we got from the last time.
2.  `ls` to list all the files. Used `wc` and `file` to estimate the 2 files.
3.  Used `diff` to find the different line which means the next level password.
4.  Copied the password and tried to log in to `bandit18` and see `Byebye!` which means that the password is correct.

[![asciicast](https://asciinema.org/a/743414.svg)](https://asciinema.org/a/743414)

**Key Takeaway:** 

**Commands Used:**
`ls`
`wc`
`file`
`diff`
---

# Level 18 -> 19

**Challenge:** The password for the next level is stored in a file readme in the homedirectory. Unfortunately, someone has modified .bashrc to log you out when you log in with SSH.

**Methodology:**
1. In this level you can't just normally log in as `bandit18` because `.bashrc` has been modified to log you out.
2. So I think if we can't log in and execute command, can we just execute command without loggin in? So I did some research and found out that you can!.
3. Used `ls` and `cat` to read the file without loggin in by using `ssh bandit18@bandit.labs.overthwire.org -p2220 ls` found `readme` file then `ssh bandit18@bandit.labs.overthwire.org -p2220 cat readme` got the password!

[![asciicast](https://asciinema.org/a/743419.svg)](https://asciinema.org/a/743419)

**Key Takeaway:** You can execute command without logging in (SSH)

**Commands Used:**
`ls`
`cat`
`ssh`
---

# Level 19 -> 20

**Challenge:** To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.

**Methodology:**
1.  Logged in as `bandit19` using the password we got from the last time.
2.  `ls -l` to list all the files and see the permissions of the file.

    ```bash
    -rwsr-x--- 1 bandit20 bandit19 14884 Aug 15 13:16 bandit20-do
    ```
    
-   The owner is badit20 and the group is bandit19 with `s` SUID user bandit19 can execute the binary, but the binary is executed as user bandit20.
  
3.  So next execute tehe setuid binary and see how to use it.
  
    ```bash
    bandit19@bandit:~$ ./bandit20-do 
    Run a command as another user.
       Example: ./bandit20-do id
    ```
    
4.  Now we have the idea how to read `/etc/bandit_pass/bandit20` which can only be read by bandit20. 

    ```bash
    bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20 
    0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
    ```

[![asciicast](https://asciinema.org/a/743877.svg)](https://asciinema.org/a/743877)

**Key Takeaway:** In addition to standard user and group permissions, Linux allows us to configure special permissions on files through the Set User ID (`SUID`) and Set Group ID (`SGID`) bits. These bits function like temporary access passes, enabling users to run certain programs with the privileges of another user or group. For example, administrators can use `SUID` or `SGID` to grant users elevated rights for specific applications, allowing tasks to be performed with the necessary permissions, even if the user themselves doesn’t normally have them. The presence of these permissions is indicated by an `s` in place of the usual `x` in the file's permission set. When a program with the SUID or SGID bit set is executed,  it runs with the permissions of the file's owner or group, rather than the user who launched it. This can be useful for certain system tasks but also introduces potential security risks if not used carefully.

**Commands Used:**
`ls`
`cat`
---

# Level 20 -> 21

**Challenge:** There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

NOTE: Try connecting to your own network daemon to see if it works as you think

**Methodology:**
1.  Logged in as `bandit20` using the password we got from the last time.
2.  `ls -l` to list all the files and see the permissions of the file.
3.  There is a setuid binary like the last level so I did `./suconnect` to see the instruction.
4.  So I think of netcat `nc` to create some kind of connection or server? and then echo the password as the first line when connection happen. So I ask AI if I can use echo and then pipe out to netcat or do I have to specify some flag to do that.
5.  So I do `echo "0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO" | nc -l -p 8080` `-l` is listening and `-p` to specify the port I think this is some kind of creating a one time server on local? Have to do some research on this.
6.  Then after pressing enter, the terminal looks like it's working so i think of job in Linux so I do `Ctrl+Z` to stop this process and then `bg 1` it to background job so I can execute other commands. I think adding `&` after the command should work the same here, but I don't know if I open another terminal and connected to bandit server and use `./suconnect 8080` going to work the same here. We got the password anyway!

[![asciicast](https://asciinema.org/a/745585.svg)](https://asciinema.org/a/745585)

**Key Takeaway:** Review the Job command in Linux (Now I know it has some use)

**Commands Used:**
`ls`
`echo`
`nc -l -p [port]`
---

# Level 21 -> 22

**Challenge:** A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

**Methodology:**
1.  Logged in as `bandit21` using the password we got from the last time.
2.  Cd to the given path `cd /etc/cron.d/` used `ls` to see all the files.
3.  So in this level let's focus on `cronjob_bandit22` let's just `cat cronjob_bandit22` to see what does this cronjob does and see what script/command/program is being executed.
  
       ```bash
       bandit21@bandit:/etc/cron.d$ cat cronjob_bandit22
       @reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
       * * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
       ```
       
    the script is executed at boot and then repeatedly once per minute, running with bandit22’s privileges and producing no visible output.

    ### Let's learn a bit about cron, cronjob, crontab
    
🔹 cron
-    A time-based job scheduler in Unix/Linux systems.
-    It runs in the background and executes tasks (scripts/commands) automatically at specified times or intervals.
-    Think of it as an “alarm clock” for commands.

🔹 cronjob
-    A task/job that you schedule to run automatically using cron.
-    Example: Backing up a database every night at midnight.

🔹 crontab (CRON TABle)
-    The configuration file where you define cronjobs.
-    Each user (including root) can have their own crontab.
-    You edit it using: `crontab -e`. You can list existing jobs with: `crontab -l`.

🔹 Crontab syntax
   
   ```bash
      * * * * *  command-to-run
      │ │ │ │ │
      │ │ │ │ └── Day of week (0-6, Sunday=0)
      │ │ │ └──── Month (1-12)
      │ │ └────── Day of month (1-31)
      │ └──────── Hour (0-23)
      └────────── Minute (0-59)
   ```

🔹 Example

-    Run a script every day at 3:30 AM: `30 3 * * * /home/user/backup.sh`

🔹In short:
    
-    cron = the scheduler (the service)
-    cronjob = the scheduled task
-    crontab = the file/table where cronjobs are defined

4.  So next let's so see the content of `/usr/bin/cronjob_bandit22.sh` do `cat /usr/bin/cronjob_bandit22.sh`

   ```bash
   bandit21@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh 
   #!/bin/bash
   chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
   cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
   ```

5.  So this file creates a file in the `tmp` folder and gives read permission to everyone. Then it copies the content of the bandit22 password file into the file.
6.  Let's just cat it `cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv`


[![asciicast](https://asciinema.org/a/745617.svg)](https://asciinema.org/a/745617)

**Key Takeaway:** man `cron`, `crontab`, `crontab(5)`

**Commands Used:**
`ls`
`cat`
---

# Level 22 -> 23

**Challenge:** A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

NOTE: Looking at shell scripts written by other people is a very useful skill. The script for this level is intentionally made easy to read. If you are having problems understanding what it does, try executing it to see the debug information it prints.

**Methodology:**
1.  Logged in as `bandit22` using the password we got from the last time.
2.  Cd to the given path `cd /etc/cron.d/` used `ls` to see all the files.
3.  `cat cronjob_bandit23` to see what does this cronjob does and see what script/command/program is being executed. The configuration shows that the script `/usr/bin/cronjob_bandit23.sh` is executed regularly as user bandit23.
4.  Then `cat /usr/bin/cronjob_bandit23.sh`
  
   ```bash
   bandit22@bandit:/usr/bin$ cat /usr/bin/cronjob_bandit23.sh 
   #!/bin/bash

   myname=$(whoami)
   mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

   echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

   cat /etc/bandit_pass/$myname > /tmp/$mytarget
   ```

-   The script calculates mytarget (the filename) using the MD5 hash of "I am user bandit23".
-   The script copies the password from /etc/bandit_pass/bandit23 to /tmp/$mytarget.

5.  At first I check if this script can be executed by bandit22 `ls /usr/bin/cronjob_bandit23.sh` and know that I can. Then I normally tried to run this script but the result was obviously going to be the password of bandit22. Then I think it has something to do with those variables but I was not sure. So I did some research.

    This level introduces how to analyze cron jobs and basic shell scripting (specifically variable assignment and command output piping).

    The crucial part of the script is how it dynamically creates a filename using a hash:

-   The script runs whoami to get the current user (bandit22) and saves it to the $myname variable.
-   It constructs a string ("I am user bandit22").
-   It pipes that string to the md5sum command, which generates a unique, fixed-length hash of the string.
-   It uses cut to clean up the hash.
-   The final password is saved to /tmp/ using this hash as the filename.

    By manually calculating this hash, we can predict the filename and retrieve the password.

6.  We need to replicate the script's hash calculation using bandit23 as the $myname variable. Then we got the file name in return.

   ```bash
   bandit22@bandit:~$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
   8ca319486bfbbc3663ea0fbe81326349
   ```

7.  Then we do `cat /tmp/8ca319486bfbbc3663ea0fbe81326349`

[![asciicast](https://asciinema.org/a/745864.svg)](https://asciinema.org/a/745864)

**Key Takeaway:** Learn more about variables in Linux.

**Commands Used:**
`ls`
`cat`
---

# Level 23 -> 24

**Challenge:** A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

NOTE: This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level!

NOTE 2: Keep in mind that your shell script is removed once executed, so you may want to keep a copy around…

**Methodology:**
1.  Logged in as `bandit23` using the password we got from the last time.
2.  Cd to the given path `cd /etc/cron.d/` used `ls` to see all the files.
3.  `cat /etc/cron.d/cronjob_bandit24` to see what does this cronjob does and see what script/command/program is being executed. The configuration shows that the script `/usr/bin/cronjob_bandit24.sh` is executed regularly as user bandit24.
4.  Then `cat /usr/bin/cronjob_bandit24.sh`
  
   ```bash
bandit23@bandit:~$ cat /usr/bin/cronjob_bandit24.sh 
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname/foo
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" ./$i)"
        if [ "${owner}" = "bandit23" ]; then
            timeout -s 9 60 ./$i
        fi
        rm -f ./$i
    fi
done
   ```

   Script Explanation

   Targeting files specifically owned by a user named bandit23 for execution with a time limit.

   Cleaning up (deleting) all files it processes, regardless of whether they were executed or not.

   The script performs the following steps:

    Sets the user name:

        myname=$(whoami) stores the current user's username in the variable myname.

    Changes directory:

        cd /var/spool/$myname/foo navigates to the directory /var/spool/current_user/foo. This is where the files/scripts to be processed are located.

    Iterates through files:

        for i in * .*; do ... done starts a loop to iterate through all files and directories in the current directory, including hidden ones (those starting with a dot, like .script.sh).

        The if [ "$i" != "." -a "$i" != ".." ]; then ... fi condition excludes the special directories . (current directory) and .. (parent directory) from processing.

    Checks file ownership and executes:

        echo "Handling $i" prints the name of the file being processed.

        owner="$(stat --format "%U" ./$i)" retrieves the owner's username of the current file ($i).

        if [ "${owner}" = "bandit23" ]; then ... fi checks if the file owner is exactly bandit23.

        If the owner is bandit23, the file is executed using timeout -s 9 60 ./$i.

            timeout -s 9 60 ensures the execution stops after a maximum of 60 seconds, killing the process with signal 9 (SIGKILL) if it runs for too long. This prevents malicious or hung scripts from running indefinitely. The script assumes these files are executable scripts.

    Deletes the file:

        rm -f ./$i unconditionally deletes the file after the check/execution. The -f flag forces the removal and suppresses most error messages.

5.  In this level, We have to write our own script but I'll just copy the past cronjob.sh script and adjust it a bit here. `mktemp -d` then `cd to that temp directory` then `vim bandit24.sh`

   ```bash
   #!/bin/bash                                                                                                                                                             
   mkdir -p /tmp/tmp.dsJ9zPh8f3/$(cat /etc/bandit_pass/bandit24)                       
   ```

6.  `chmod +x bandit24.sh`
7.  Then do `cp bandit24.sh /var/spool/bandit24/foo/`
8.  After that we have to set perms so that the system can execute our script. `chmod 777 /tmp/tmp.dsJ9zPh8f3`
9.  Wait for a few minutes for the cronjob to do its job. And the password will appear.

**Key Takeaway:** Learn more about Scripting in Linux.

**Commands Used:**
`chmod`
`cat`
---

# Level 24 -> 25

**Challenge:** A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.
You do not need to create new connections each time

**Methodology:**
1.  Logged in as `bandit24` using the password we got from the last time.
2.  Looks like we have to make a script that will automatically tried all the 10000 pincode combination plus netcating to that port?.
3.  First, Let's `mktemp -d` for writing a script then connected to the port using netcat to see the script and its response so we know how we will write the script.
  
    ```bash
    bandit24@bandit:/tmp/tmp.ZIkWkXX1BT$ nc localhost 30002
    I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
    w
    Wrong! Please enter the correct current password and pincode. Try again.
    ```
    
4.  We know that's it one-liner response so let's write a simple script for brute-forcing `vim bruteforce.sh`

   ```bash
   #!/bin/bash

   for i in {0000..9999}; do
           echo (your bandit24 password) $i >> possib.txt
   done

   cat possib.txt | nc localhost 30002 > result.txt
   ```

### Script Explanation
   The script attempts to guess a password by systematically trying every four-digit number from `0000` to `9999` and submitting it to a server.
---
1\. Generating the Payload (Creating `possib.txt`)

```bash
for i in {0000..9999}; do
    echo (your bandit24 password) $i >> possib.txt
done
```

  * **`for i in {0000..9999}; do ... done`**: This loop iterates through all numbers from $0$ to $9999$. The brace expansion ensures the numbers are padded with **leading zeros** (e.g., $0001, 0010$).
  * **`echo (your bandit24 password) $i`**: For each number \`$i$, it prints a required phrase (a placeholder in the example) followed by the four-digit number.
  * **`>> possib.txt`**: This **appends** the generated line to a file named `possib.txt`.

The result is a file containing $10,000$ lines, where each line is a potential password attempt.

---

 2\. Attacking the Service (Submitting the Guesses)

```bash
cat possib.txt | nc localhost 30002 > result.txt
```

  * **`cat possib.txt`**: The `cat` command reads the contents of the generated file (`possib.txt`).
  * **`|` (Pipe)**: The output of `cat` is piped as input to the next command.
  * **`nc localhost 30002`**: The `nc` (netcat) utility is used to establish a **TCP connection** to the server running on the **local machine (`localhost`)** at **port `30002`**.
  * The $10,000$ lines from `possib.txt` are sent one by one over this network connection to the service. This is the **brute-force attempt**.
  * **`> result.txt`**: The response from the network service (which is usually the server's output, including success or failure messages) is redirected and saved to a file named `result.txt`. The user would then inspect `result.txt` to find the correct password line.
    
5.  Then we just execute tehe script `bash bruteforce.sh`.
6.  Then `cat result.txt | grep -v "Wrong!"` because we know the pattern will respond `Wrong!` if it wrong so we do `grep -v "Wrong!"` to exclude all the lines that contain `Wrong!`

[![asciicast](https://asciinema.org/a/746704.svg)](https://asciinema.org/a/746704)

**Key Takeaway:** Learn more about Scripting in Linux.

**Commands Used:**
`nc`
`vim`
`cat`
`grep -v`
---

# Level 25 -> 26, 26 -> 27

**Challenge:** Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not /bin/bash, but something else. Find out what it is, how it works and how to break out of it.

NOTE: if you’re a Windows user and typically use Powershell to ssh into bandit: Powershell is known to cause issues with the intended solution to this level. You should use command prompt instead.

**Methodology:**
1.  Logged in as `bandit25` using the password we got from the last time.
2.  In this level I struggled a lot so I looked up a walkthrough which is https://mayadevbe.me/posts/overthewire/bandit/level26/ which explain everything you need to know.
3.  So after obtained the password of bandit26 by making a terminal smallest while connecting(ssh) to bandit26 using the sshkey provided in the bandit25 session, I finally connected to bandit26 in More command and then press `v` to go into Vim. Then set the shell by `:set shell=/bin/bash` then use `:shell` to get into command line mode in vim and just `cat /etc/bandit_pass/bandit26`, we got the password!. I also do `ls` while first successfully be able to use command line in Vim, I found 2 files. First, is the ASCII text that was being showed, and the other is the script which make execute command as bandit27 so I do `./bandit27-do cat /etc/bandit_pass/bandit27` We got bandit27 password.

[![asciicast](https://asciinema.org/a/746973.svg)](https://asciinema.org/a/746973)

**Key Takeaway:** Learn more about More and Vim especially Command Mode. (More command does have some presence in my life...)

**Commands Used:**
`more`
`vim`
`cat`
`ssh -i`
---

# Level 27 -> 28

**Challenge:** There is a git repository at ssh://bandit27-git@localhost/home/bandit27-git/repo via the port 2220. The password for the user bandit27-git is the same as for the user bandit27.

Clone the repository and find the password for the next level.

**Methodology:**
1.  Logged in as `bandit27` using the password we got from the last time.
2.  Make a temporary directory for cloning the repo `mktemp -d` and `cd` into it.
3.  First, I tried normal git clone because I've seen people used this command so I'll just try it. `git clone ssh://bandit27-git@localhost/home/bandit27-git/repo` but it doesn't work. I think it's because I didn't specify the port, as seen in the instruction and the response from the server.
4.  So I scrolled through `man git` and found `GIT_SSH_COMMAND` so I quickly looked up this command in google. Gemini answer how can I do it so I did `GIT_SSH_COMMAND='ssh -p2220' git clone ssh://bandit27-git@localhost/home/bandit27-git/repo` and I successfully cloned the repo.
5.  Use `ls` to list all the files in the repo. Found `README.md` just like in github so I just `cat` it and I got the password.

[![asciicast](https://asciinema.org/a/747287.svg)](https://asciinema.org/a/747287)

**Key Takeaway:** Learn more about Git 

   ```bash
   >$ whatis git
   Git (3pm)        - Perl interface to the Git version control system
   git (1)          - the stupid content tracker
   ```

**Commands Used:**
`git`
`git clone`
`GIT_SSH_COMMAND`
`cd`
`ls`
`cat`
`mktemp -d`
---

# Level 28 -> 29

**Challenge:** There is a git repository at ssh://bandit28-git@localhost/home/bandit28-git/repo via the port 2220. The password for the user bandit28-git is the same as for the user bandit28.

Clone the repository and find the password for the next level.

**Methodology:**
1.  Logged in as `bandit28` using the password we got from the last time.
2.  Make a temporary directory for cloning the repo `mktemp -d` and `cd` into it.
3.  Just do what we did at the level 27->28. It's the same process here. Clone the repo with `GIT_SSH_COMMAND` to specify the port
4.  Found `README.md` like the same so I `cat` it, but the content is pretty weird.
  
    ```bash
    bandit28@bandit:/tmp/tmp.rgf8fgsvFe/repo$ cat README.md 
    # Bandit Notes
    Some notes for level29 of bandit.

    ## credentials

    - username: bandit29
    - password: xxxxxxxxxx
    ```
   
6.  So I did `git log` (I have seen this command before) and saw that there are a few of commits. So I start from the first one.
7.  At first, I did not know which command to use so I did `man git` and `git --help` and luckily found the right command which is `git show`
8.  So I did `git show 710c14a2e43cfd97041924403e00efb00b3a956e` and I found the password that got deleted.

[![asciicast](https://asciinema.org/a/747719.svg)](https://asciinema.org/a/747719)

**Key Takeaway:** Learn more about Git Log. 

**Commands Used:**
`git`
`git clone`
`git log`
`git show`
`GIT_SSH_COMMAND`
`cd`
`ls`
`cat`
`mktemp -d`
---

# Level 29 -> 30

**Challenge:** There is a git repository at ssh://bandit29-git@localhost/home/bandit29-git/repo via the port 2220. The password for the user bandit29-git is the same as for the user bandit29.

Clone the repository and find the password for the next level.

**Methodology:**
1.  Logged in as `bandit29` using the password we got from the last time.
2.  Make a temporary directory for cloning the repo `mktemp -d` and `cd` into it.
3.  Just do what we did at the level 27->28. It's the same process here. Clone the repo with `GIT_SSH_COMMAND` to specify the port
4.  Cat `README.md` like the same, but the content says like it's in production and i also check log and by the name it's really nothing. I know that GitHub has branches so I assume that Git would have too. Git branching is another feature of the version control system. It allows you to split the development into different branches.
5.  So I do `git branch -la` to list all branches and found there are some branches so I start from the top and fortunately it's the one. I think you can take the clue that in master brach README.md says that the password is in production which implies it's in development.
6.  So I do `git checkout dev` to switch to that branch.
7.  When I was in, I just normally do `ls` found README.md and just `cat README.md` and there's the password!

[![asciicast](https://asciinema.org/a/748174.svg)](https://asciinema.org/a/748174)

*Key Takeaway:** Learn more about Git Branching. 

**Commands Used:**
`GIT_SSH_COMMAND`
`git`
`git clone`
`git branch -a`
`git checkout`
`cd`
`ls`
`cat`
`mktemp -d`
---

# Level 30 -> 31

**Challenge:** There is a git repository at ssh://bandit30-git@localhost/home/bandit30-git/repo via the port 2220. The password for the user bandit30-git is the same as for the user bandit30.

Clone the repository and find the password for the next level.

**Methodology:**
1.  Logged in as `bandit30` using the password we got from the last time.
2.  Make a temporary directory for cloning the repo `mktemp -d` and `cd` into it.
3.  Just do what we did at the level 27->28. It's the same process here. Clone the repo with `GIT_SSH_COMMAND` to specify the port
4.  Cat `README.md` like the same, but the content says It's empty, so I checked branches and there is nothing also logs too. So it took me a while, to find the right command.
5.  `git tag` Git tagging is a way to mark specific points in the history of the repository. One example would be to mark release points of the software.
6.  So I did `git tag` to show all the tags. Found `secret` tag.
7.  After that, just do `git show secret` and got the password!

[![asciicast](https://asciinema.org/a/748408.svg)](https://asciinema.org/a/748408)

*Key Takeaway:** Learn more about Git Tagging. 

**Commands Used:**
`GIT_SSH_COMMAND`
`git`
`git clone`
`git tag`
`git show`
`cd`
`ls`
`cat`
`mktemp -d`
---

# Level 31 -> 32

**Challenge:** There is a git repository at ssh://bandit31-git@localhost/home/bandit31-git/repo via the port 2220. The password for the user bandit31-git is the same as for the user bandit31.

Clone the repository and find the password for the next level.

**Methodology:**
1.  Logged in as `bandit31` using the password we got from the last time.
2.  Make a temporary directory for cloning the repo `mktemp -d` and `cd` into it.
3.  Just do what we did at the level 27->28. It's the same process here. Clone the repo with `GIT_SSH_COMMAND` to specify the port
4.  Cat `README.md` like the same, and the content says that we have to add files with specific content and push it.
5.  So first I did `echo "May I come in?" > key.txt`.
6.  Check the `.gitignore` and found that `.txt` files will be ignore unless we delete this .gitignore or use `-f` to force add when using `git add`
7.  So I `rm ./.gitignore` then `git add key.txt` to update that this file will be part of the next commit.
8.  Next do `git commit` to saves the currently made changes with a message describing these changes. Nano will pop up and just add your comment, save it and exit nano.
9.  This steps really piss me but you don't have to feel what felt. Just do `GIT_SSH_COMMAND='ssh -p2220 git push` and we got the password!

[![asciicast](https://asciinema.org/a/748412.svg)](https://asciinema.org/a/748412)

*Key Takeaway:** Learn more about Git Tagging. 

**Commands Used:**
`GIT_SSH_COMMAND`
`git clone`
`git add`
`git commit`
`git push`
---

# Level 32 -> 33

**Challenge:** After all this git stuff, it’s time for another escape. Good luck!

**Methodology:**
1.  Logged in as `bandit32` using the password we got from the last time.
2.  I have to admit that this level confused me a lot. I had to look up a walkthrough online...
3.  So I'll drop the walkthrough here (https://mayadevbe.me/posts/overthewire/bandit/level33/) read the walkthrough and all the comment section. Really help!
4.  I think I am not qualified to write a walkthrough on this level yet... So I leave it here. Just a mark on this level that I have done it but not fully understand.

[![asciicast](https://asciinema.org/a/748611.svg)](https://asciinema.org/a/748611)

*Key Takeaway:** Learn more about Variables and Shell Variables. 

**Commands Used:**
`$0`
`cat`
`ssh`
`man`
---

# Level 33 -> 34

At this moment, level 34 does not exist yet.
