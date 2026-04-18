---
title: "Letter - TryHackMe"
date: 2026-04-17 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [osint, newspaper, 23-may-1925-disaster, barcode]
image: /assets/img/writeups/covers/letter.png
---
# Letter - TryHackMe
![1.png](/assets/img/writeups/letter/1.png)
**Category:** OSINT<br>**Difficulty:** Medium

> Warning: Full solution ahead. Try the room yourself first!

---

## Overview

This room is a pure OSINT challenge. The goal is to dig through historical records, decode a French postal barcode, and track down the identity of a young crew member from a 1925 maritime disaster in Brittany.<br><br>It's a genuinely fun room that teaches you how to approach open source research methodically, especially when the clues are buried in old newspaper archives and French historical websites.

---

## Part 1 — Finding the Postal Code
![1.png](/assets/img/writeups/letter/letter-newspaper.png)
The room gives you a set of asset files to download and examine. One of them is an image of an old envelope. The text on it is heavily wrinkled and stained but there's something interesting on it — a row of orange bars:

```
..||||| |.||.| ||..|| |||..| .||.||
```

These are a **French postal barcode**, used by the French postal service to encode address information on letters. The bars represent digits and help mail sorting machines read addresses automatically, even when the handwriting is hard to read.<br><br>To decode it, you need this chart which I sourced from Wikipedia:

| Value | Coding |
|-------|--------|
| 0 | ..\|\|\|\| |
| 1 | .\|.\|\|\| |
| 2 | .\|\|.\|\| |
| 3 | .\|\|\|.\| |
| 4 | \|..\|\|\| |
| 5 | \|.\|.\|\| |
| 6 | \|.\|\|.\| |
| 7 | \|\|..\|\| |
| 8 | \|\|.\|.\| |
| 9 | \|\|\|..\| |

Now place the barcode segments next to the table and decode them one by one:

```
..||||| → 0 (start bar, ignore)
|.||.|  → 6
||..||  → 7
|||..|  → 9
.||.||  → 2
```

This gives us: `bar + 06792` which written together is `06792`.<br><br>One important thing to know about French postal codes is they are read **right to left**. So flipping `06792` gives us:<br><br>**Postal Code: `29760`**

---

## Part 2 — Finding the Flag

The flag format the room gives us is:

```
THM{Name_Surname_age}
```

Only the first letter of the name and surname should be capitalised. So we need to find a real person's name, surname, and age.<br><br>The room also gives you a `note.txt` file. Here's the translation:

> *"My dear Édouard, Today, while tidying the attic at my grandparents' house, I came across this old newspaper clipping. Your great-grandfather wasn't even old enough to get his driver's license when he distinguished himself that day. The youngest of the team, and certainly not the least courageous. He would be so proud to see you on the water too. With all my love, Audette"*

Two things stand out immediately. First, whoever we're looking for was the **youngest member of a team**. Second, he was **not old enough to have a driver's license**, which typically means he was under 18. Keep both of those in mind.

---

### Step 1 — Identifying the Newspaper and the Story
![1.png](/assets/img/writeups/letter/3.png)
The asset files include a newspaper clipping. I began on searching for keywords from the newspaper to see what it is talking about, one of the headlines visible on it mentions **Amunda-t-il atteint le pole Nord?**.
![1.png](/assets/img/writeups/letter/4.png)
A Google search confirms that the newspaper is talking about **Did Amunda reach the North Pole?** As we are in future, Google confirms that Amundsen successfully reached the North Pole on **12 May 1926** as part of the Norge expedition. After reading some articles and history about the Roald Amunda I came to know that he made earlier attempts — one in `1918-1925` and one in `1925` where his crew went missing and was feared dead.
![1.png](/assets/img/writeups/letter/6.png)
The newspaper seems to be from around **1925** not 1926, because that's when the disappearance story was making headlines. The newspaper in the room is **L'Ouest-Éclair**, a French regional paper from Brittany.

---

### Step 2 — Narrowing Down the Date

Manually flipping through newspaper archives would take hours. This is where using AI as a research tool makes sense. Ask it something like:

> *"On which dates did L'Ouest-Éclair cover the disappearance of Roald Amundsen in 1925?"*

![1.png](/assets/img/writeups/letter/7.png)
The AI points us toward **22–24 May 1925** as the window when the story was being covered.<br><br>I searched for `L'Ouest-Éclair` newspaper archives and found a website. I set the date to around 22–24 May 1925 and found the exact newspaper we were provided with.
![1.png](/assets/img/writeups/letter/5.png)
![1.png](/assets/img/writeups/letter/8.png)

The specific date was **24 May 1925**, which is when L'Ouest-Éclair covered the disappearance story. But while digging around that date range, another story catches the eye — dated **23 May 1925**.
![1.png](/assets/img/writeups/letter/date.png)

---

### Step 3 — The Real Story

The 24 May article contains this (translated from French):

> *"AUDIERNE, May 23. — The launch Sainte-Barbe, with a crew of seventeen men, was returning to port during a storm around 3:00 PM. At the end of the pier, heavy waves swamped the boat. All the men were thrown into the sea. Two were rescued by the Audierne station lifeboat, and the others by their own means except for one who drowned."*

This is the incident we're looking for. A boat disaster off the coast of Brittany involving a crew of seventeen men. One didn't make it.


---

### Step 4 — Finding the Crew on KBC PENMARC'H

To get more detail about this incident and the names of crew members, we need to go deeper. Searching around the story leads to a French historical research website called **KBC PENMARC'H** (KBC ENMARC'H), which is dedicated to the history and archives of Penmarc'h, a commune in the Finistère department of Brittany.<br><br>It took a few minutes to find the incident. It was located in the site's dropdown sections, clearly labeled as:<br><br>**DISASTER OF MAY 23, 1925**

![1.png](/assets/img/writeups/letter/9.png)
![1.png](/assets/img/writeups/letter/10.png)
Read through the full article. It lists the crew members involved in the disaster. Going back to what the note told us — youngest member, under 18 — scan the crew list for the youngest person.
![1.png](/assets/img/writeups/letter/11.png)
There he is:<br>**GOURLAOUEN (Yves-Marie) — 15 years old**<br><br>He was the youngest crew member on board that day and the note's description fits him perfectly. A 15-year-old on a fishing boat crew during a storm in 1925 is exactly the kind of story a grandparent would be remembered for. Later he was awarded the Silver Medal, 2nd Class, by the French government.

---

### Assembling the Flag

We have everything:

- **Name:** Yves-Marie
- **Surname:** Gourlaouen
- **Age at the time:** 15

Following the flag format `THM{Name_Surname_age}` with first letters capitalised:

```
THM{Yves-Marie_Gourlaouen_15}
```

---

## What This Room Teaches You

This room is a great example of how real OSINT investigations actually work. You start with an envelope, notice the orange bars, decode a French postal barcode using a Wikipedia chart, and work out the postal code. Then a handwritten note points you toward a newspaper clipping, a headline about Amundsen gives you a date range, and from there you dig into French newspaper archives to land on a specific maritime disaster from 23 May 1925. When the archives don't give you enough detail about the crew, you track down a regional French historical website dedicated to Penmarc'h and find the crew list buried in a dropdown section. One name, one age, and the flag assembles itself.


# A Quick Note

I'm not a historian and some of the historical details in this writeup like the events, dates, and people involved may not be 100% accurate. I did my best to research and present the information correctly but if anything is wrong I apologise for that.<br><br>The main purpose of this writeup is simply to help you solve TryHackMe's **Letter** room. If you spot any historical inaccuracies feel free to reach out and I'll correct them.

Happy hacking!

---

*Written by monasx0 | TryHackMe — Letter*
