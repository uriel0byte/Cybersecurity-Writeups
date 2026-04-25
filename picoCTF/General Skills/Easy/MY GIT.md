# Author: Darkraicg492

# Description
I have built my own Git server with my own rules! Additional details will be available after launching your challenge instance.

# Hints
1.  How do you specify your Git username and email?

# Steps
1. Use `git clone` command to clone the remote repository.
2. Use `cat` command to see the README.md file.
```
# MyGit

### If you want the flag, make sure to push the flag!

Only flag.txt pushed by ```root:root@picoctf``` will be updated with the flag.

GOOD LUCK!
```
3. Use `git config user.name "root"` and `git config user.email "root@picoctf"` to specify Git username and email.
4. Use `git config user.name` and `git config user.email` to verify the settings.
5. Then I read the README again and it said that "Only flag.txt pushed by..." but this file doesn't exist, so we have to create one.
6. Use `touch flag.txt` to create and empty file name flag.txt
7. Use `git add flag.txt` to put updated file in a box.
8. Use `git commit -m "Give me the flag"` to tape the box shut.
9. Use `git push` to upload the box to GitHub. Put the password in.
10. The update show up with the flag.
```
urielbyte-picoctf@webshell:~/challenge$ git push
git@foggy-cliff.picoctf.net's password: 
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 2 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 257 bytes | 257.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Author matched and flag.txt found in commit...
remote: Congratulations! You have successfully impersonated the root user
remote: Here's your flag: picoCTF{1mp3rs0n4t4_g17_345y_f3a6488d} <---------------------(SEE THIS?)
To ssh://foggy-cliff.picoctf.net:56579/git/challenge.git
   4142dd3..e59b08f  master -> master
```

Answer: picoCTF{1mp3rs0n4t4_g17_345y_f3a6488d}
