---
title: "Cipher's Secret Message - TryHackMe"
date: 2026-04-07 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [cipher, secret, cryptography]
image: /assets/img/writeups/covers/ciphers-secret-message.png
---
# Cipher's Secret Message - TryHackMe
![1.png](/assets/img/writeups/ciphers-secret-message/1.png)
**Platform:** TryHackMe<br>**Difficulty:** Easy<br>**Time Required:** 10 min
## Overview
This room is a beginner-friendly introduction to basic cryptography. The main goal is to understand how hidden messages (ciphers) work and how to decode them.
![1.png](/assets/img/writeups/ciphers-secret-message/2.png)
## Understanding the Encryption Algorithm
````
from secret import FLAG

def enc(plaintext):
    return "".join(
        chr((ord(c) - (base := ord('A') if c.isupper() else ord('a')) + i) % 26 + base) 
        if c.isalpha() else c
        for i, c in enumerate(plaintext)
    )

with open("message.txt", "w") as f:
    f.write(enc(FLAG))
````

This code takes a secret message called FLAG, encrypts it using a custom method, and saves the result into a file. The encryption works by going through each letter in the message and shifting it forward in the alphabet based on its position (first letter stays the same, second shifts by 1, third by 2, and so on). It also makes sure uppercase and lowercase letters stay in their correct ranges, while non-letter characters remain unchanged.<br><br>**Rule**
- Each letter was shifted forward by its position (i)
- So to decrypt, we shift backward by i
## Step-by-step Decoding First Chunk: a_up4qr

We go character by character:
###  a / position 0
- `Shift back by 0 → stays a`
### _  / position 1
- `Not a letter → stays _`
###  u / position 2
Shift back by 2
- `u → t → s → becomes s`
###  p / position 3
Shift back by 3
- `p → o → n → m → becomes m`
###  4 / position 4
- `Not a letter → stays 4`
###  q / position 5
Shift back by 5
- `q → p → o → n → m → l → becomes l`
###  r / position 6
Shift back by 6
- `r → q → p → o → n → m → l → becomes l`


**Result of first chunk:**
````
a_sm4ll
````
After manually decoding the first chunk, the pattern becomes clear. Each character is shifted backward based on its position in the string, while non-alphabet characters remain unchanged.<br><br>With this understanding, it becomes straightforward to decode the rest of the message.<br><br>The final Flag is:
````
THM{a_sm4ll_crypt0_message_to_st4rt_with_THM_cracks}
````
