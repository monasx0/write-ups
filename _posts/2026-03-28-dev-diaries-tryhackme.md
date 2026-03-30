---
title: "Dev Diaries - TryHackMe"
date: 2026-03-29 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [osint, github, ctf, crt.sh]
image: /assets/img/writeups/covers/dev-diaries.png
---
# Dev Diaries - TryHackMe
![1.png](/assets/img/writeups/dev-diaries/1.png)
**Difficulty**: Easy<br>**Time Required**: 30 min<br>[**Dev Diaries**](https://tryhackme.com/room/devdiaries) is an easy-level TryHackMe room that focuses on OSINT techniques. The objective is to gather publicly available information from sources such as certificate transparency logs, website content, and Git commit history to uncover hidden details and retrieve flags.<br>**🔧Tools Used**: Browser, crt.sh, GitHub<br>**⚠️ Warning**: This write-up contains full solutions and flag locations. Attempt the room yourself before reading further.
## Finding the Subdomain
Navigate to [crt.sh](https://crt.sh/) and search for `marvenly.com`. Certificate transparency logs will reveal all subdomains associated with this domain.
![1.png](/assets/img/writeups/dev-diaries/2.png)
## Finding the Github Username
Visit the discovered subdomain and scroll to the Contact section at the bottom of the page. The developer's GitHub username is publicly disclosed there.
![1.png](/assets/img/writeups/dev-diaries/3.png)
## Finding the Developer's Email
Clone the repository and inspect the logs using:
```
$ git log
````
Alternatively, navigate to the repository's Commits section on GitHub, open the second most recent commit, and append `.patch` to the URL. The developer's email address will be visible in the commit details. 

![1.png](/assets/img/writeups/dev-diaries/4.png)
## Reason for Source Code Removal
The reason behind the source code removal is documented within the same commit details used in the previous task. Look at the subject line of the commit message for the answer.
![1.png](/assets/img/writeups/dev-diaries/5.png)
## Finding the Hidden Flag
Return to the same commit used in the previous tasks. Scroll to the very bottom of the commit details — the hidden flag is located at the end of the commit content.
![1.png](/assets/img/writeups/dev-diaries/6.png)
