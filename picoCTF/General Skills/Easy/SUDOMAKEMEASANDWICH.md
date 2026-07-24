# Author: Darkraicg492

# Description
Can you read the flag? I think you can!

# Hints
1. What is sudo?
2. How do you know what permission you have?

# Steps
1. Launch an instance, connect to the remote server via SSH with the provided credentials.
2. List all the files in the landing directory found flag.txt but the permissions on the files is available for root user and root group.
3. Used id and sudo -l commands to evaluate which user am I logged in to and groups belong to this account and see which command or binary I can use it with sudo.
4. I see "emacs" which is a text editor.
5. Execute "sudo emacs flag.txt".

Answer: picoCTF{ju57_5ud0_17_d8e1a280}
