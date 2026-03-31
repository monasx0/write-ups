---
title: "Insp3ct0r - picoCTF"
date: 2026-03-29 02:00:00 +0000
categories: [picoCTF, Writeups]
tags: [js, css, html, inspect]
image: /assets/img/writeups/covers/picoctf.png
---
# Insp3ct0r - picoCTF
**Category**: Web Exploitation<br>**Difficulty**: Easy
![1.png](/assets/img/writeups/picoctf/insp3ct0r/1.png)
## Overview
This challenge presents a webpage and requires finding a flag split across three parts, each hidden inside a different file — HTML, CSS, and JavaScript. The goal is to use the browser's Developer Tools to inspect each file and piece the flag together.

## Opening Developer Tools
Navigate to the target URL and open your browser's Developer Tools by pressing F12 or right-clicking anywhere on the page and selecting Inspect.

### Flag Part 1 (HTML)
Go to the Elements tab in Developer Tools. Scroll to the very bottom of the HTML source. The first part of the flag is hidden there in a comment.
![1.png](/assets/img/writeups/picoctf/insp3ct0r/2.png)

### Flag Part 2 (CSS)
Navigate to the Sources tab and open the stylesheet (.css file). Scroll to the bottom of the file. The second part of the flag is hidden in a comment at the end.
![1.png](/assets/img/writeups/picoctf/insp3ct0r/3.png)
### Flag Part 3 (JavaScript)
Still in the Sources tab, open the JavaScript (.js) file. Scroll to the bottom. The third and final part of the flag is hidden in a comment there as well.
![1.png](/assets/img/writeups/picoctf/insp3ct0r/4.png)
Combine all three parts in order and sumbit the flag.
