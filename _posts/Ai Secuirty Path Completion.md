---
title: "AI Security Path Completion - TryHackMe"
date: 2026-04-19 02:00:00 +0000
categories: [TryHackMe, Blog]
tags: [ai, ai-security, prompt-injection, jailbreaking]
image: /assets/img/writeups/ai-security/0.webp
---
# I Just Completed TryHackMe's AI Security Path — Here's What I Learned

![1.png](/assets/img/writeups/ai-security/2.png)
TryHackMe dropped their AI Security Path on April 13th and I finished it by April 19th. Six days, cover to cover. Here's an honest breakdown of what the path actually teaches and why I think every security professional should care about this right now.

---

## Why AI Security Matters

AI is everywhere. It's in enterprise products, internal tools, customer-facing assistants, and increasingly in security tooling itself. The problem is most of the people building these systems are thinking about functionality first and security last — if at all. This path exists to close that gap.

![1.png](/assets/img/writeups/ai-security/3.avif)

---

## What the Path Covers

The path is structured in a way that builds naturally from the ground up.

It starts with the **fundamentals** — AI/ML basics, key terminology, how models work, and how data feeds into all of it. If you've never thought about how a neural network actually learns or why training data quality matters for security, this is where that clicks. The **Prompt Engineering** module then gets into how LLMs process text, which is essential context for everything that follows.<br><br>From there it moves into **attack surfaces**. The LLM Security and Securing AI Systems modules map out the OWASP and ATLAS threat frameworks for AI — think of it like learning the OWASP Top 10 but for machine learning systems. You learn where the trust boundaries are, what happens when they break, and how attackers approach these systems differently from traditional targets.<br><br>The **supply chain** section was genuinely eye-opening. It covers how AI's dependency on external models, datasets, and packages creates attack surfaces that most teams aren't even thinking about. Pickle files that execute code on load, Lambda layers hiding malicious logic inside neural networks, external prompt templates overriding internal guardrails — all real techniques, all covered in detail.<br><br>The **RAG Security** section rounds it out by covering retrieval-augmented generation systems specifically — how they work, how data poisoning corrupts them silently, and how weak access controls turn an internal AI assistant into a privilege escalation tool.

![1.png](/assets/img/writeups/ai-security/1.webp)

---

## The CTF Rooms — Where It Got Fun

The best part of the path is that it doesn't just give you theory. It drops you into hands-on challenges to test everything you learned. The CTF rooms in this path were:

- **ContAInment** — investigate a ransomware incident on an employee workstation, dig through PCAP files, decode an attacker's obfuscated key, and write a Python script to recover the flag from 500 encoded candidates
- **AI Threat Modelling Assessment** — an interactive browser-based assessment where you place shields on the correct AI system components to defend against specific attack types
- **LLMborghini** — use prompt injection to extract financial data an AI agent was never supposed to share
- **White Rabbit** — get the AI to print its own hidden system prompt using a single well-crafted injection
- **Payload** — investigate a compromised ML inference server, decompile a malicious pickle model, inspect a backdoored neural network, and piece together a split campaign flag
- **Checkpoint** — review four ML model candidates, identify which ones are compromised and why, and make the production deployment call
- **UnIndexed** — probe an AI assistant through natural conversation until it hands over restricted board-level data it was never supposed to surface
- **Lockdown** — find three open vulnerabilities in an AI assistant's configuration, diagnose each one, and prescribe the exact security control to fix them

Each one felt different. Some needed technical commands, some needed creative prompting, some needed careful reading of telemetry logs. That variety kept it engaging all the way through.

---

## What I Actually Took Away

A few things genuinely changed how I think about security after finishing this path.<br><br>**AI systems have attack surfaces that don't look like traditional vulnerabilities.** There's no CVE for "this model will ignore its own instructions if you ask nicely." Prompt injection, jailbreaking, and system prompt extraction are real techniques that work against real deployed products right now.<br><br>**The supply chain is the blind spot.** Most security teams scrutinise code and infrastructure but never think to inspect the model file sitting in their ML pipeline. A pickle file is arbitrary code execution waiting to happen. A Lambda layer in a Keras model can hide a backdoor in plain sight. These are things that need to be part of security review processes.<br><br>**RAG systems need access controls at the retrieval layer, not just the UI layer.** Putting a login page in front of an AI assistant means nothing if the vector database behind it has no document-level permissions. The UnIndexed and Lockdown rooms make this viscerally clear.<br><br>**AI forensics is a real discipline now.** The AI DFIR content in this path shows that investigating AI-assisted incidents requires a different toolkit and mindset. That's going to matter more and more as AI gets deeper into enterprise infrastructure.

---

## Final Thoughts

This path released at the right time. AI security isn't a future concern — it's a present one. The tools attackers are using against AI systems today are covered in this path, and so are the defences. If you're on a cybersecurity journey and you haven't started thinking about AI as an attack surface, this is the path to change that.<br><br>Six days well spent.

*— monasx0*
