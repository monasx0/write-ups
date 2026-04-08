---
title: "Meow - HackTheBox"
date: 2026-04-08 02:00:00 +0000
categories: [HackTheBox, Writeups]
tags: [openvpn, telnet, nmap]
image: /assets/img/writeups/covers/hackthebox.jpeg
---
# Meow - HackTheBox
![1.png](/assets/img/writeups/hack-the-box/Meow/1.png)
**Platform:** HackTheBox<br>**Tier:** 0 · Very Easy · Linux <br>**Author:** monasx0  

---

## Introduction

After spending time sharpening my skills on TryHackMe CTFs, I decided to give HackTheBox a try. On my first day, the **Meow** challenge from the Starting Point appeared — an approachable beginner box designed to get comfortable with the HTB environment.<br>

---

## Guided Questions

### Q1 — What does the acronym VM stand for?

**Answer: Virtual Machine**<br>

A software-based emulation of a physical computer. HTB uses VMs as both the attacker and target environment.

---

### Q2 — What tool do we use to interact with the OS via command line — also called a console or shell?

**Answer: Terminal**<br>

The terminal is where everything happens in CTFs — running tools, reading output, catching flags. I use it every single day.

---

### Q3 — What service do we use to form our VPN connection into HTB labs?

**Answer: OpenVPN**<br>

Just like TryHackMe, HTB requires a VPN tunnel to reach lab machines. You download a `.ovpn` config file and connect with:

```bash
openvpn <your-file.ovpn>
```

---

### Q4 — What tool do we use to test our connection to the target with an ICMP echo request?

**Answer: Ping**<br>

Before running any heavy recon tools, a quick ping confirms the machine is alive and reachable — a habit from every CTF.

```bash
ping <TARGET-IP>
```

---

### Q5 — What is the most common tool for finding open ports on a target?

**Answer: Nmap**<br>

The industry-standard network scanner. Used for discovering open ports, running services, OS fingerprinting, and much more.

---

### Q6 — What service do we identify on port 23/tcp during our scans?

**Answer: Telnet**<br>

Port 23 is the well-known port for Telnet — an older, unencrypted remote access protocol. Seeing it open is immediately a red flag in a CTF environment.

---

### Q7 — What username is able to log into the target over Telnet with a blank password?

**Answer: root**

In misconfigured or default setups, the root account sometimes has no password set — giving direct administrative access. This is exactly the kind of misconfiguration that earns findings in real-world bug bounty programs too.

---

## Flag Capture

After confirming port 23 is open with `nmap`, we connect via Telnet using `root` with no password. Once inside the shell, the flag is retrieved with a simple `cat` command.


```bash
telnet <TARGET-IP>
```

![1.png](/assets/img/writeups/hack-the-box/Meow/2.png)
![1.png](/assets/img/writeups/hack-the-box/Meow/3.png)

---

## Quick Reference

| Question | Answer |
|---|---|
| VM stands for | Virtual Machine |
| Command-line interface tool | Terminal |
| VPN service used | OpenVPN |
| Connectivity test tool | Ping |
| Port scanning tool | Nmap |
| Service on port 23 | Telnet |
| Username with blank password | root |

---
