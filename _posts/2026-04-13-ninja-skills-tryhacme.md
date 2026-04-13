---
title: "Ninja Skills - TryHackMe"
date: 2026-04-13 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [linux, find, grep, regex]
image: /assets/img/writeups/covers/ninja-skills.png
---
# Ninja Skills - TryHackMe

![1.png](/assets/img/writeups/ninja-skills/0.png)
**Category:** Linux Challenges
**Difficulty:** Easy

> ⚠️ **Warning:** Full solutions ahead. Try the room yourself first!

---

## Overview

This room is all about mastering the `find` command on Linux. Each question gives you a list of files scattered across the entire filesystem and asks you to extract specific information about them. The trick is combining `find` with other tools like `grep`, `sha1sum`, and `wc` to answer each question in a single efficient command.

**The files we're working with throughout this room:**

```
8V2L  bny0  c4ZX  D8B3  FHl1  oiMO  PFbD  rmfX  SRSq  uqyw  v2Vb  X1Uy
```

Since these files are scattered all over the system, every command starts with a `find` that searches the entire filesystem (`/`) for all of them at once. We'll build on top of that base throughout.

---

## Q1 — Which files are owned by the `best-group` group?

```bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -group best-group 2>/dev/null
```
![1.png](/assets/img/writeups/ninja-skills/1.png)
**Flags explained:**

| Flag | What it does |
|---|---|
| `find /` | Start searching from the root of the filesystem |
| `-type f` | Only look for regular files (ignore directories, symlinks, etc.) |
| `\( ... \)` | Group multiple conditions together |
| `-name 8V2L` | Match files with this exact name |
| `-o` | Logical OR — match this name **or** the next one |
| `-group best-group` | Filter results to only files owned by the group `best-group` |
| `2>/dev/null` | Suppress "Permission denied" errors so output stays clean |

---

## Q2 — Which file contains an IP address?

This one needs two commands. First we find the IP, then we find which file it came from.

**Step 1 — Find the IP address:**

```bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -exec grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" {} \; 2>/dev/null
```
![1.png](/assets/img/writeups/ninja-skills/2.png)
**Step 2 — Find which file contains it:**
Just append the `-H` flag to the previous command.

```bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -exec grep -E -o -H "([0-9]{1,3}[\.]){3}[0-9]{1,3}" {} \; 2>/dev/null
```
![1.png](/assets/img/writeups/ninja-skills/3.png)
**New flags explained:**

| Flag | What it does |
|---|---|
| `-exec ... {} \;` | For each file found, run the command that follows. `{}` is replaced by the file path |
| `grep -E` | Use Extended Regular Expressions (needed for patterns like `{1,3}`) |
| `-o` | Print only the matching part, not the whole line |
| `"([0-9]{1,3}[\.]){3}[0-9]{1,3}"` | Regex pattern that matches an IPv4 address (four groups of 1–3 digits separated by dots) |
| `-H` | Print the filename alongside the match — this is what reveals which file contains the IP |

---

## Q3 — Which file has the SHA1 hash `9d54da7584015647ba052173b84...`?

```bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -exec sha1sum {} \; 2>/dev/null
```
![1.png](/assets/img/writeups/ninja-skills/4.png)
This runs `sha1sum` on every file and prints each file's hash alongside its path. Just scroll through the output and match the hash from the question.

**New flag explained:**

| Flag | What it does |
|---|---|
| `sha1sum` | Computes the SHA1 cryptographic hash of a file |

---

## Q4 — Which file contains 230 lines?

```bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -exec wc -l {} \; 2>/dev/null
```
![1.png](/assets/img/writeups/ninja-skills/5.png)
This prints the line count for every file found.

> 📝 **Note:** All 11 files that are listed will show a line count of 209. The one file that doesn't appear in the output at all is your answer — that's the file with 230 lines.

**New flag explained:**

| Flag / Part | What it does |
|---|---|
| `wc -l` | Counts the number of lines in a file |

---

## Q5 — Which file's owner has a user ID of 502?

```bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -user 502 2>/dev/null
```
![1.png](/assets/img/writeups/ninja-skills/6.png)
Instead of searching by username, we search directly by **user ID (UID)**. Linux identifies users internally by numbers — `502` maps to a specific user account on this machine.

**New flag explained:**

| Flag / Part | What it does |
|---|---|
| `-user 502` | Match files owned by the user with UID `502` |

---

## Q6 — Which file is executable by everyone?

```bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) -exec ls -l {} + 2>/dev/null
```
![1.png](/assets/img/writeups/ninja-skills/7.png)
This lists all the files with their full permissions. In the output, look at the permission string on the left — for example:

```
-rwxrwxr-x
```

A file executable by **everyone** (owner, group, and others) will have `x` in all three permission groups. It will look like:

```
-rwxrwxr-x   ← owner=rwx, group=rwx, others=r-x 
```

**New flags explained:**

| Flag / Part | What it does |
|---|---|
| `ls -l` | List files in long format, showing permissions, owner, group, and size |
| `{} +` | Similar to `\;` but passes all found files to the command at once (faster than running one by one) |

---

*Written by monasx0 | TryHackMe — Ninja Skills*
