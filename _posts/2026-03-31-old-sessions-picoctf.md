---
title: "Old Sessions - picoCTF"
date: 2026-03-31 02:00:00 +0000
categories: [picoCTF, Writeups]
tags: [sessions, cookie, ctf, flag]
image: /assets/img/writeups/covers/picoctf.png
---
# Old Sessions - picoCTF
**Category**: Web Exploitation<br>**Difficulty**: Easy
![1.png](/assets/img/writeups/picoctf/old-sessions/1.png)
[Old Sessions](https://play.picoctf.org/practice/challenge/739?category=1&page=1) is a beginner-friendly picoCTF challenge focused on session management and authentication. The goal is to analyze how session data is handled and reuse or manipulate an existing session to gain unauthorized access. It highlights common web security issues related to improper session handling and teaches the importance of secure session management.
## Finding the Hidden Flag
First, create a new account on the target machine with any username and password.
![1.png](/assets/img/writeups/picoctf/old-sessions/2.png)
Then log in with the same username and password.
![1.png](/assets/img/writeups/picoctf/old-sessions/3.png)
The key observation is that a user mentioned discovering a page at `/sessions`. Visiting this endpoint reveals the session token associated with the `admin` user.
![1.png](/assets/img/writeups/picoctf/old-sessions/4.png)
Now, copy the identified session token and replace your current session token using the browser’s developer tools. Navigate to the **Application tab**, locate the Cookies section for the target domain, and update the value of the session cookie with the copied token. Once modified, refresh the page to apply the changes and access the authenticated session.
![1.png](/assets/img/writeups/picoctf/old-sessions/5.png)
Now, return to the homepage and refresh the page to retrieve the flag.
![1.png](/assets/img/writeups/picoctf/old-sessions/6.png)
