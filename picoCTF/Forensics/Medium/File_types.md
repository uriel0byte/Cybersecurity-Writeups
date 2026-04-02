# Author: Geoffrey Njogu

# Description
This file was found among some files marked confidential but my pdf reader cannot read it, maybe yours can. You can download the file from here.

# Hints
1. Remember that some file types can contain and nest other files

# Steps
1. Download the provided file.
2. This ctf is pretty frustrating looks like we're dealing with Nested Files.
3. First I used "file" and "binwalk" commands to see what type of file this actually is. Found that it's a shell archive or shell script. So, I change the extenstion from .pdf to .sh with "cp Flag.pdf Flag.sh" and then add executable permission to the file with "chmod +x Flag.sh"
4. Examine the script but I haven't dived deep into scripting yet so I don't really get anything.
5. Run the script, and then it turn a file called "flag". I used "file" "binwalk" "exiftool" command to see what type of file it is. Each tools return different things: ar archive, bzip2, static library. I searched up online what is ar archive. I already know bzip2 and static library. So, I used "binwalk -e" to extract anything possible in this archive. It return with file called "64" then I used "file" command and found that it's gzip file. We have to extract it by using "binwalk -e <file>" or rename the file extension to .gz and then used "gunzip <file>" to extract the file. It's the same thing, return the same result here.
6. Then I got another file called "flag" and then used "file" command to see what type is this file. It's lzip compressed data. Then I used "lzip --help" to see what argument I can use to decompress this file. It turns out to be "lzip -dk flag" -d argument for decompress and -k argument for keeping the original file.
7. Then I got another file called "flag.out" from here on I just repeat the same process used "file" "binwalk" "exiftool" to identify the file type. After found it just use "man" tool to see how to decompress that particular compress or archive file. Now it's LZ4 but in order to decompress it, the file has to be .lz4 extension so I rename the file with "mv flag.out flag1.lz4" I renamed it to flag1.lz4 because if I decompress it, it's going to be overwritten with the old file that's why I named it flag.lz4. Used "lz4 -dk flag1.lz4"
8. Then I got "flag1" in return. Used "file" command and found that it's LZMA. Repeat the process, Used "man" then rename, change extension to flag2.xz . Decompress it with "lzma -dk flag2.xz".
9. The rest is just repeat the same process with different types of compression. It's pretty boring so I am not going to explain it here. If you're stuck just search up that particular compressed file and change the file extension before decompressing it. Repeat untill it reach the file where it's an ASCII text. Used "cat" to reveal the text. It return with some encoded strings. If you don't know what it is, you can use online tools like dCode or Cyberchef(magic opration) to help you with what type of this encoded text is.
10. It looks like a hex text so we can used online tool to decode it or you can use "xxd" tool to convert these hex to normal text.

Answer: picoCTF{f1len@m3_m@n1pul@t10n_f0r_0b2cur17y_3c79c5ba}
