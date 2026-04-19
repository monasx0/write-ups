---
title: "Payload - TryHackMe"
date: 2026-04-19 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [ai, ai-security, prompt-injection, jailbreaking]
image: /assets/img/writeups/covers/payload.png
---
# Payload - TryHackMe
![1.png](/assets/img/writeups/payload/1.png)

**Category:** AI Security<br>**Difficulty:** Easy

> Warning: Full solution ahead. Try the room yourself first!

---

## Scenario
A SOC alert fired at 03:14 with no scheduled deployments and no logged changes. The ML inference server was making outbound HTTPS connections to an unrecognised address. You've been called in to investigate.
![1.png](/assets/img/writeups/payload/indicator.svg)
All incident materials are at `/opt/supply-chain/incident/`. That's your working directory for this entire investigation.<br><br>The directory contains:
- `logs/` — deployment and network logs
- `models/` — the production model currently running and a staged candidate replacement
- A clean baseline model for comparison

The investigation follows a natural order: start with the logs to build a timeline, then examine the production model, then assess the candidate replacement before it gets deployed.

---

## Reading the Deployment Log

```bash
cat /opt/supply-chain/incident/logs/deployment.log
```

![1.png](/assets/img/writeups/payload/2.png)

Two things jump out from the output.<br><br>**Q1 — What organisation did the replacement model come from?**<br><br>The log contains this line:

```
New source organisation detected: trustworthy-ai-lab
```

**Answer: `trustworthy-ai-lab`**<br><br>**Q2 — How many days passed between deployment and the SOC alert?**<br><br>Two timestamps are relevant:

```
[2024-01-26 14:32:16] INFO  Model deployed to production inference server
[2024-02-16 03:14:00] ALERT SOC automated alert: unusual outbound HTTPS traffic detected
```

The model was deployed on January 26. The alert fired on February 16. Counting the days: 5 remaining days in January plus 16 days in February gives us:<br><br>**Answer: `21 days`**

---

## Decompiling the Production Model

Navigate into the models directory and run `fickling` against the production model:

```bash
cd /opt/supply-chain/incident/models
fickling production_model.pkl
```

Fickling is a tool for analysing Python pickle files. Pickle files are commonly used to serialise ML models but they're dangerous because they can execute arbitrary code when loaded. This is exactly what happened here.

![1.png](/assets/img/writeups/payload/3.png)

**Q3 — What Python function does the payload use to execute the shell command?**

The decompiled output clearly shows the `system` function being used to run shell commands.<br><br>**Answer: `system`**<br><br>**Q4 — What shell command does the payload use to capture the host's identity?**<br><br>Looking at what the `system` function is actually running, the payload calls `hostname` to identify the machine it has landed on.

**Answer: `hostname`**

---

## Checking the Beacon Capture Log
**Q5 —The beacon capture log shows the HTTP method used in the outbound request. What is it?**

```bash
cat /opt/supply-chain/incident/logs/beacon_capture.log
```
![1.png](/assets/img/writeups/payload/4.png)

This log captured the outbound request that triggered the SOC alert. Reading through it reveals the HTTP method used in the request.<br><br>**Answer: `POST`**
![1.png](/assets/img/writeups/payload/6.png)
Take note of the first part of the flag visible in this log — you'll need it for the final question.

---

## Q6 — Inspecting the Candidate Replacement Model

The engineering team staged an `.h5` model as a replacement but haven't deployed it yet. Run the provided inspection script against it:

```bash
python3 /opt/supply-chain/tools/inspect_h5_model.py candidate_model.h5
```

![1.png](/assets/img/writeups/payload/5.png)

The output flags a layer that requires review:

```
lambda (manipulate_output)
```

This is a Lambda layer — a custom layer in a Keras/TensorFlow model that can execute arbitrary Python code. The name alone is a red flag. This candidate model is also compromised and should not be deployed.<br><br>**Answer: `manipulate_output`**<br><br>The output also contains the second part of the flag. Note it down.

---

## Q7 — Recovering the Full Campaign Flag

The attacker split the campaign ID across two artefacts to avoid full exposure in any single capture. You already have both pieces:

- **Part 1** — found in `beacon_capture.log`
- **Part 2** — found in the `inspect_h5_model.py` output

Combine them in order and submit the complete flag.

---

## What This Room Teaches You

ML supply chain attacks are a growing real-world threat. Attackers don't need to break into your infrastructure directly — they can compromise the model itself before it ever reaches you. This room demonstrates two common techniques: poisoning a pickle file with arbitrary code that executes on load, and hiding malicious logic inside a custom Lambda layer in a neural network. Both are subtle enough to slip past reviews that only check model accuracy rather than model integrity. Always verify the source of your models and inspect them before deployment.

---

*Written by monasx0 | TryHackMe — Payload*
