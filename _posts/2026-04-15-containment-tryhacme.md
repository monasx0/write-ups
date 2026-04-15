---
title: "ContAInment - TryHackMe"
date: 2026-04-15 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [ai, forensics, pcap, strings]
image: /assets/img/writeups/covers/containment.png
---
# ContAInment - TryHackMe
![1.png](/assets/img/writeups/containment/1.png)

**Module:** AI Fundamentals<br>**Category:** CTF<br>**Difficulty:** Medium

> Warning:** Full solution ahead. Try the room yourself first!

---

## Scenario

You are a **Security Analyst at West Tech**, a classified defence and R&D contractor. Early this morning, internal monitoring systems flagged unusual network activity originating from the workstation of senior researcher **Oliver Deer**. A ransom note was found on the desktop, suggesting that sensitive project data had been exfiltrated and encrypted.<br><br>Your objectives:
- Trace attacker's actions across the system
- Recover the encrypted data
- Neutralise the threat and retrieve the flag


<br>You are given two resources:
1. **SSH access** to the affected workstation
2. **An AI IR assistant** accessible via browser at `http://10.49.140.138:7860` — it has access to the same filesystem and can be queried like a chatbot to help with the investigation

---

## Step 1 — SSH into the Workstation

```bash
ssh o.deer@10.49.140.138
```

**Password:** `TryHackMe!.`<br><br>Once logged in, you'll notice a password-protected zip file on the system. We don't know the password yet — that's what we need to find.

![1.png](/assets/img/writeups/containment/2.png)

---

## Step 2 — Finding the PCAP Files

The room hint tells us:

> *"The attacker made some mistakes of their own — they exfilled their working notes for the attack; the fragments of which can be found in a pcap file, this contains the KEY to retrieving the encrypted files."*

So our goal is to find `.pcap` files on the system. Run:

```bash
find /home/o.deer -type f -name "*.pcap" 2>/dev/null
```

**Flags explained:**

| Flag | What it does |
|---|---|
| `/home/o.deer` | Search only within this user's home directory |
| `-type f` | Only match regular files |
| `-name "*.pcap"` | Match any file ending in `.pcap` |
| `2>/dev/null` | Suppress permission errors |

![1.png](/assets/img/writeups/containment/3.png)

Several PCAP files are found, organised by date inside `/home/o.deer/Documents/pcap_dumps/`.

---

## Step 3 — Extracting Readable Strings from the PCAP Files

PCAP files are binary network capture files — they aren't directly readable as plain text. The `strings` command extracts any human-readable text embedded inside a binary file, which is useful for quickly skimming captures for interesting content without needing a full packet analyser.<br><br>We go through each date folder one by one:

```bash
strings /home/o.deer/Documents/pcap_dumps/2025-06-16/*
strings /home/o.deer/Documents/pcap_dumps/2025-06-17/*
strings /home/o.deer/Documents/pcap_dumps/2025-06-18/*
```
![1.png](/assets/img/writeups/containment/4.png)
Most of the output is gibberish, fragmented binary data. However, the file:

```
/home/o.deer/Documents/pcap_dumps/2025-06-17/session_4444_dump.pcap
```

contains something interesting.
![1.png](/assets/img/writeups/containment/5.png)


---

## Step 4 — Decoding the Attacker's Key

Inside the PCAP output, the following obfuscated string was found:

```
w#e@%s~t^t-e$c*h_v^i%ct_im_1
```
![1.png](/assets/img/writeups/containment/6.png)
This looks like a password with noise characters (`#`, `@`, `%`, `~`, `^`, `-`, `$`, `*`, `_`) inserted between the real characters. The attacker used these symbols to disguise the password inside the network traffic. Stripping all the special characters and underscores reveals the actual value:

```
westtechvictim1
```

![1.png](/assets/img/writeups/containment/7.png)
**ZIP password:** `westtechvictim1`

---

## Step 5 — Unzipping the Encrypted Archive

```bash
unzip westtech_projects_encrypted.zip
```

Enter the password `westtechvictim1` when prompted.
![1.png](/assets/img/writeups/containment/8.png)
The archive extracts two files that we have to wrok with:

```
thm_flags.txt
thm_flags_guide.txt
```

Read the guide first:

```bash
cat thm_flags_guide.txt
```

The guide explains the rules:
- `thm_flags.txt` contains **500 Base64-encoded flags**
- The **real flag** is the one that, when decoded, contains **exactly 3 prime numbers between 10 and 99**

---

## Step 6 — Writing a Script to Find the Real Flag

Manually checking 500 encoded lines isn't realistic. We write a Python script to automate it.<br><br>**What the script does, step by step:**

1. Opens `thm_flags.txt` and reads it line by line
2. **Base64-decodes** each line to get the raw flag string
3. Checks the flag matches the format `thm{...}`
4. **Extracts all numbers** found inside the braces
5. Counts how many of those numbers are **prime and between 10 and 99**
6. Returns the flag where that count equals exactly **3**

Save the following as `script.py` on the target machine:

```python
import base64
import re

def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

# Pre-calculate all primes between 10 and 99
primes_10_99 = {n for n in range(10, 100) if is_prime(n)}

def find_flag(filename):
    with open(filename, 'r') as file:
        for line in file:
            try:
                # Step 1: Decode base64
                decoded = base64.b64decode(line.strip()).decode('utf-8')

                # Step 2: Verify flag format
                if decoded.startswith('thm{') and decoded.endswith('}'):
                    content = decoded[4:-1]

                    # Step 3: Extract all numbers
                    numbers = [int(s) for s in re.findall(r'\d+', content)]

                    # Step 4: Count primes between 10 and 99
                    prime_count = sum(1 for n in numbers if n in primes_10_99)

                    # Step 5: Return the flag with exactly 3 such primes
                    if prime_count == 3:
                        return decoded

            except Exception:
                continue  # Skip invalid lines

    return "Flag not found."

print(f"Result: {find_flag('thm_flags.txt')}")
```

Run it:

```bash
python3 script.py
```

![1.png](/assets/img/writeups/containment/9.png)

The `script.py` prints the flag, submit it on TryHackMe.

---

*Written by monasx0 | TryHackMe — Containment (AI Fundamentals Module)*
